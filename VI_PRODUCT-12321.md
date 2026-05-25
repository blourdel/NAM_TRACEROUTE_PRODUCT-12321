# VI PRODUCT-12321 — NAM Traceroute capability expansion

- Jira: [PRODUCT-12321](https://dt-rnd.atlassian.net/browse/PRODUCT-12321)
- Area: NAM synthetic monitors (Traceroute)
- Status: Draft (Use Case Defined target)
- Last updated: 2026-05-23

> Structured against the VCG official 2023 VI template. Values marked
> *(proposed)* are working assumptions to unblock Use Case Defined review;
> confirm with PM, PA, and Customer Zero before sign-off.
>
> *Scope note*: This VI is exclusively about the NAM Traceroute synthetic
> test. Probe protocols within traceroute (ICMP, UDP, TCP) are in scope.


## Short Abstract / Blogline

Expand the NAM Traceroute synthetic test with multi-protocol probes
(ICMP, UDP, TCP), per-hop enrichment, path-change detection, and
historical path diff, so operators can resolve path-level network
issues using NAM data alone — at parity with Datadog Network Path and
SolarWinds NetPath.

## Customer Zero
- *Primary*: Dynatrace internal NetOps/SRE team operating production
  NAM Traceroute monitors across multi-region infrastructure. They need
  hop-visibility, path-change, and ICMP-blocked-path gaps captured in
  the WHY section.

Customer Zero validates: traceroute multi-protocol behavior, per-hop
enrichment accuracy, path-change event quality, alert quality under
low sample counts.


## Customer One
 A number of customers are already using the traceroute extension. They will be approached to be design partners of NAM traceroute. They cts as external validation during Public Preview.

 Potential customer list
 
| Name | Dynatrace contact |
| --- | --- |
| HKG | Derek Choi |
| Tata | Bharat Singh |
| Cardinal Health | David Detoro |
| Mass General Brigham | Tim Lowell |
| Ausgleichskasse des Kantons Bern | Florian Buehler |


## Target Audience

- Network Operations Engineers troubleshooting reachability and
  path-level incidents.
- Site Reliability Engineers correlating path-level signals during
  incident triage and postmortems.
- Platform Administrators managing large fleets of NAM traceroute
  monitors and rollout policies.


## WHY

Ping or TCP NAM  validates that a target is reachable along *some*
path but stops short of the diagnostic depth operators need for modern
network troubleshooting. As a result, investigations escalate to
external tools (mtr, traceroute, tcptraceroute, packet captures) or
to competitor products (Datadog Network Path, SolarWinds NetPath) that
already provide multi-protocol probes, per-hop enrichment, path-change
detection, and alerting. MTTR grows, deals are lost on head-to-head
capability questions, and customers question whether NAM is sufficient
for network synthetic monitoring.

### Where it hurts

- *No hop-level visibility worth using*: Existing Extension traceroute
  output is not integrated with in the platform and is not integrated
   in the Synthetic app, making it difficult to manage.
- *No path-change detection*: Routing changes (ISP failover, BGP
  reconvergence, MPLS re-routes) silently affect performance and NAM
  has no signal for them.
- *No hop enrichment*: Hops show as raw IPs without reverse DNS or
  ASN/owner labels, so operators cannot tell whose network they are
  looking at.
- *ICMP-blocked paths*: Enterprise firewalls and many ISPs commonly
  drop or rate-limit ICMP, leaving classic ICMP-only traceroute 
  provide by the traceroute extension blind.
  Competitors default to TCP-based path discovery for this reason.
- *No native path alerting*: Without per-hop and path-change
  thresholds, traceroute is diagnostic-only, so operators don't keep
  monitors enabled and lose the proactive signal.
- *Alert noise risk*: Sparse sample sizes on per-hop metrics will
  produce false positives unless thresholds are sample-size-aware.

External tooling and competitor products fill these gaps today, which
fragments the operator workflow and weakens the case for NAM as the
primary network synthetic platform.

## Traceroute 101

This section is background for non-networking readers and for anyone
who needs to understand why probe-protocol choice matters in this VI.

### How traceroute works in general

Every IP packet has a Time-To-Live (TTL) field that each router on the
path decrements by one. When TTL reaches zero, the router drops the
packet and returns an ICMP *Time Exceeded* message to the sender,
revealing the router's IP. Traceroute exploits this by sending a
sequence of packets with TTL = 1, 2, 3, … and recording which router
replies at each step, building an ordered list of hops between source
and destination. The destination itself is identified by a different,
protocol-specific reply (see below). Latency for each hop is the
round-trip time of that probe.

Three things differ between protocol variants: what kind of packet the
probe is, how the *final* hop (the destination) confirms it received
the probe, and how friendly the protocol is to firewalls, ISPs, and
load balancers on the way.

### ICMP-probe traceroute

The classic Windows `tracert`. Probes are ICMP Echo Request packets;
intermediate hops reply with ICMP Time Exceeded; the destination
replies with ICMP Echo Reply.

- *Pros*:
  - Simplest to implement and easiest to reason about.
  - No destination port to choose.
  - Works well on the open public Internet where ICMP is generally
    allowed.
  - Symmetric with `ping`, so operators already understand the
    semantics.
- *Cons*:
  - Enterprise firewalls and many ISPs rate-limit or drop ICMP, so
    paths through corporate networks frequently end in `* * *`.
  - Some routers deprioritize ICMP generation in software, producing
    *inflated per-hop latency* that does not reflect the latency a
    real TCP/UDP flow would see.
  - Cannot prove that the actual service port on the destination is
    reachable — only that the destination IP is.
  - Anycast and load-balanced destinations may answer ICMP from a
    different instance than the one that would handle production
    traffic.

### UDP-probe traceroute

The classic Unix default (`traceroute`). Probes are UDP packets sent
to a high, typically-closed destination port (the original choice was
33434 and incrementing); intermediate hops reply with ICMP Time
Exceeded; the destination replies with ICMP *Port Unreachable* because
nothing is listening on that port.

- *Pros*:
  - Default on most Unix-like systems and on many appliances, so the
    output is familiar to operators.
  - Varying the destination port across probes lets the tool
    influence ECMP hashing and discover *multiple* equal-cost paths
    (the "Paris / Dublin traceroute" family of techniques).
  - Often gets further through firewalls than ICMP, because the
    outbound UDP is usually allowed and the return ICMP errors are
    treated as related traffic.
- *Cons*:
  - Final-hop detection depends on ICMP Port Unreachable, which is
    rate-limited on many destinations and can be filtered entirely.
  - High UDP destination ports are sometimes dropped at strict
    enterprise egress firewalls.
  - Still cannot prove the *service* port is reachable, only that the
    destination is.
  - Some networks rate-limit UDP more aggressively than TCP, so
    per-hop loss can be misleading.

### TCP-probe traceroute

The basis of `tcptraceroute` and the default mode of SolarWinds
NetPath. Probes are TCP SYN packets sent to a specific service port
(commonly 80 or 443); intermediate hops reply with ICMP Time Exceeded;
the destination replies with TCP SYN-ACK (port open) or TCP RST (port
closed). Either reply confirms the destination is reachable on that
port.

- *Pros*:
  - Most likely protocol to traverse strict enterprise firewalls,
    because outbound TCP to common service ports is almost always
    allowed.
  - Tests reachability of the *actual service port*, not just the IP,
    so the path discovered matches what production traffic uses.
  - Final-hop signal (SYN-ACK or RST) is not subject to ICMP
    rate-limiting on the destination.
  - Lets operators detect mid-path TCP-aware middleboxes (proxies,
    transparent terminators) that would intercept production flows.
- *Cons*:
  - Implementation requires raw sockets or elevated privileges,
    which constrains where the runner can be deployed.
  - SYN probes can trip IDS/IPS rules that flag port-scan behavior;
    coordination with security teams may be needed for sensitive
    targets.
  - The probe must pick a destination port up front; choosing the
    wrong one can produce a misleading path or be flagged as scanning.
  - TCP-terminating middleboxes (some load balancers, CDN edges) may
    answer SYN-ACK on behalf of the real destination, shortening the
    observed path.

### When to use which (operator's rule of thumb)

| Path / scenario                                | Recommended probe                       |
|------------------------------------------------|-----------------------------------------|
| Open public Internet, ICMP allowed             | ICMP                                    |
| Inside or across enterprise firewalls          | TCP (to the service port)               |
| Need to discover ECMP / multi-path             | UDP (vary port) or TCP (vary port)      |
| Validate that the actual service is reachable  | TCP (to the service port)               |
| ICMP rate-limited or dropped on path           | TCP, fall back to UDP                   |
| Diagnosing latency a real flow would see       | TCP (matches production hashing best)   |

This VI commits to supporting all three probe modes with automatic
fallback so operators do not have to pick the "right" protocol up
front — see *Functional Requirements / Solution Journey* and the
parity matrix below.

## Competitive Landscape

Datadog and SolarWinds set the de-facto bar for traceroute and
path-analysis capability. NAM must reach parity to remain credible in
head-to-head deals and to remove the operator workflows that currently
bridge to external tools for hop-level diagnostics.

### Datadog — Network Performance Monitoring and Network Path

- Hop-by-hop path analysis between source agent and destination
  endpoint, run continuously on a schedule (not just ad-hoc).
- Multi-protocol probes (ICMP, UDP, TCP) with automatic fallback so
  paths resolve even when intermediate devices rate-limit or drop ICMP.
- Per-hop latency, packet loss, and jitter with sample-size context.
- Reverse DNS and ASN/owner enrichment on every hop.
- Cloud topology overlay — hops on AWS, Azure, and GCP are labelled
  with region, VPC, and managed-service identifiers.
- Path-change detection with historical diff and replay.
- Native alerting on path change, per-hop latency, and per-hop loss.
- Tight integration with Datadog APM, Synthetics, and Service Map for
  end-to-end correlation.


  Blog: https://www.datadoghq.com/blog/network-test-protocols/

### SolarWinds — NetPath, NPM, Engineer's Toolset

- TCP-based path discovery as the *default* mode (NetPath), because
  ICMP is dropped or rate-limited on many enterprise and ISP paths;
  UDP and ICMP modes available as alternatives.
- Hop-by-hop latency, packet loss, and jitter per probe interval.
- ASN and owner identification per hop, plus network-layer segmentation
  (LAN, WAN/ISP, cloud, target-service zone) shown visually.
- Continuous probing from multiple polling agents to the same target,
  with cross-agent comparison.
- Path-change tracking with historical replay and snapshot diff.
- Alerting on path change, hop latency, and hop loss; integrated into
  NPM and Server & Application Monitor.
- Engineer's Toolset adds ad-hoc traceroute and related utilities for
  on-demand investigation outside of scheduled monitoring.

### Parity Matrix

`NAM target this VI` = parity items committed to in this VI's scope.
`future VI` = explicitly deferred to a sibling VI to keep this one
shippable. `partial` = exists today but does not meet operator
expectations versus the competitive bar.

| Capability                                  | Datadog   | SolarWinds   | Dynatrace today | NAM target this VI |
|---------------------------------------------|:---------:|:------------:|:---------:|:------------------:|
| ICMP-probe traceroute                       | yes       | yes          | Extension  | yes                |
| TCP-probe traceroute                        | yes       | yes (default)| no        | yes                |
| UDP-probe traceroute                        | yes       | yes          | no        | yes                |
| Automatic probe-protocol fallback           | yes       | partial      | no        | yes                |
| Per-hop latency                             | yes       | yes          | Extension   | yes                |
| Per-hop packet loss                         | yes       | yes          | no        | yes                |
| Per-hop jitter                              | yes       | yes          | no        | yes                |
| Reverse DNS enrichment per hop              | yes       | yes          | no        | yes                |
| ASN / owner enrichment per hop              | yes       | yes          | no        | yes                |
| Network-layer segmentation (LAN/WAN/cloud)  | partial   | yes          | no        | yes                |
| Continuous scheduled path probing           | yes       | yes          | Extension  | yes                |
| Configurable max-hops and probe count       | yes       | yes          | Extension  | yes                |
| Path-change detection                       | yes       | yes          | Extension | yes                |
| Historical path diff / replay               | yes       | yes          | no        | yes                |
| Native alerting on path change              | yes       | yes          | no        | yes                |
| Native alerting on per-hop loss / latency   | yes       | yes          | no        | yes                |
| Cloud topology overlay (AWS/Azure/GCP)      | yes       | partial      | no        | future VI          |
| Ad-hoc on-demand probe tooling              | partial   | yes (toolset)| partial   | future VI          |

### Implications for this VI

- *Probe-protocol parity is non-negotiable*: TCP-probe traceroute is
  the default in SolarWinds NetPath because ICMP is rate-limited or
  blocked across many enterprise paths. NAM must support TCP, UDP, and
  ICMP probe modes — with automatic fallback — or lose deals where
  operators need to traverse strict firewalls.
- *Hop enrichment is the differentiator*: Both competitors enrich every
  hop with reverse DNS and ASN/owner. Without it, NAM traceroute output
  reads as "raw IPs" to operators and fails the eye test in side-by-side
  demos.
- *Path-change detection drives alert value*: Without it, traceroute is
  diagnostic-only. With it, traceroute becomes a proactive signal worth
  keeping monitors enabled for, which sustains adoption.
- *Deferred items kept the VI shippable*: Cloud-topology overlay and
  ad-hoc toolset-style utilities are excluded so the core parity bar
  is reachable in a single release cycle; sibling VIs cover them.

### Competitive Acceptance Criteria

In addition to the per-capability items in *E2E Acceptance Criteria*,
the VI is competitively complete when:

1. *Side-by-side review*: For three representative customer scenarios
   (long-haul WAN, ICMP-blocked enterprise path, multi-cloud egress),
   NAM Traceroute produces diagnostic output equivalent to or better
   than Datadog Network Path and SolarWinds NetPath on the
   parity-matrix items committed to this VI.
2. *Field positioning*: SE enablement materials include a competitive
   positioning section with the parity matrix and one head-to-head
   demo per competitor.
3. *No "go elsewhere" gaps*: No item that the parity matrix marks
   `yes` for both Datadog and SolarWinds is missing from NAM at GA.
   Cloud topology overlay and ad-hoc tooling are excluded by the
   explicit deferrals documented in *Out of Scope*.

## Functional Requirements / Solution Journey

### Use Cases

1. NetOps engineer configures a Traceroute monitor against a critical
   service, selects the probe protocol (ICMP, UDP, or TCP) per path
   conditions, and gets a continuously updated hop-by-hop view with
   reverse-DNS and ASN/owner labels on every hop.
2. SRE receives an alert when the path to a critical service changes
   (new hop sequence, new ASN appears, or per-hop latency/loss crosses
   a threshold) and opens the historical path diff to see exactly
   which hops changed and when.
3. Platform admin rolls out the expanded traceroute capability with
   safe defaults so existing traceroute monitors continue to run
   unchanged, then switches selected monitors to TCP-probe mode where
   ICMP is blocked.
4. SRE sets thresholds on per-hop loss and per-hop latency with
   deterministic behavior under low sample counts (for example
   "evaluate only when sample size ≥ N").
5. Investigator opens a historical traceroute execution and side-by-side
   diffs it against the previous execution and the last known good
   path to confirm a routing change as root cause of an incident.
6. Analyst queries historical NAM data to trend per-hop metrics and
   path-change frequency across regions and correlate with incidents.


### User Stories

- *NetOps engineer (multi-protocol)*: "As a NetOps engineer, I want a
  scheduled traceroute that runs over TCP probes when ICMP is blocked,
  so I can see the path through firewalled segments without external
  tcptraceroute."
- *Incident commander (path alerting)*: "As an incident commander, I
  want to be alerted the moment the path to a Tier-1 service changes
  hop sequence or ASN, so I know about ISP/peering issues before
  customers report them."
- *Investigator (path diff)*: "As an investigator, I want a
  side-by-side diff of the current path against the last known good
  path, so I can see exactly which hops changed and stop pasting
  traceroutes into spreadsheets."
- *Operator (hop enrichment)*: "As an operator, I want each hop
  labelled with reverse DNS and ASN/owner, so I immediately know
  whose network the latency is in."
- *Platform admin*: "As a platform admin, I want safe defaults for
  the new options, so my existing traceroute monitor fleet keeps
  running without a forced migration."
- *Alerting owner*: "As an alerting owner, I want sample-size-aware
  thresholds, so one hop blip doesn't page on-call."
- *Analyst*: "As an analyst, I want per-hop metrics and path-change
  events available in DQL and dashboards, so I can correlate regional
  path trends with incidents."


### Solution Concept

1. Extend the NAM Traceroute runner to a multi-protocol implementation
   supporting ICMP, UDP, and TCP probes with automatic fallback when
   an upstream protocol is rate-limited or blocked.
2. Extend monitor schema and API to expose new options — probe
   protocol, max-hops, per-hop probe count, per-hop timeout,
   path-change sensitivity — with backward-compatible defaults.
3. Extend the data pipeline and storage schema so each execution is
   stored as an ordered sequence of hop records, with per-hop
   latency/loss/jitter time series queryable in DQL.
4. Add a hop-enrichment pipeline that resolves reverse DNS and looks
   up ASN/owner per hop IP, with caching to keep per-execution cost
   low.
5. Add a path-comparison service that diffs each new execution against
   a rolling "last known good" path and emits a path-change event
   when hop sequence, ASN sequence, or layer segmentation changes
   meaningfully.
6. Surface new fields in the monitor configuration UI 
   with inline guidance, including a hop-by-hop visualization with
   layer segmentation (LAN / WAN / ISP / cloud / target zone) and a
   path-diff view; ship recommended thresholds for alerting templates,
   including path-change alerts.
7. Document the new options, migration guidance, competitive
   positioning (parity matrix), and troubleshooting patterns; ship
   release notes and update help.

### Delivery Phases

- *Phase 1 — Foundation*: Finalize metric and hop-record taxonomy with
  PA; implement multi-protocol runner and data-model updates; extend
  API surface behind a feature flag.
- *Phase 2 — Integration*: Hop-enrichment pipeline; path-comparison
  service and path-change events; UI configuration and result-view
  additions; alerting template updates; dashboard and DQL compatibility;
  scale validation.
- *Phase 3 — Rollout*: Internal Adoption → Private Preview (design
  partner) → Public Preview → GA. Defaults finalized and feature flag
  flipped at GA.

### Dependencies (informational — link in Jira)

- *NAM runner team*: Multi-protocol Traceroute runner (ICMP / UDP /
  TCP) with fallback; max-hops and per-hop probe-count handling.
- *Hop-enrichment service*: reverse DNS resolver and ASN/owner lookup
  pipeline with caching. Confirm whether an existing internal service
  can be reused or a new lightweight component is needed.
- *Path-comparison service*: diffing engine and path-change event
  emission. Likely a new service co-located with the NAM result
  pipeline.
- *Data platform*: schema evolution for hop sequences and per-hop time
  series; ingest cost review; retention policy for expanded metrics
  including per-hop history.
- *API platform*: contract review, changelog entry, versioning policy
  confirmation for the expanded traceroute payload shape.
- *UI/UX*: monitor configuration form and result-view additions
  including hop-by-hop visualization with layer segmentation,
  path-diff view, and competitive-positioning visuals; design QA
  before GA.
- *Alerting platform*: support for path-change events and per-hop
  thresholds in the existing alerting surface.
- *Documentation / Field enablement*: help pages, release note,
  competitive positioning deck, head-to-head demo scripts.


## Non-Functional Requirements

- *Performance*: similar to ICMP and TCP NAM; new metric emission and hop
  enrichment stay within an agreed per-run resource budget.
- *Reliability*: Result completeness and protocol-fallback behavior
  consistent across retries and partial samples.
- *Scalability*: Schema and pipeline support existing and projected
  traceroute monitor volume — including the per-hop time series —
  without breaking ingestion SLOs.
- *Usability*: New options (probe protocol, max-hops, probe count,
  path-change sensitivity) are understandable to a NetOps generalist;
  defaults are safe.

## Enablement Requirements

- *Support*: Runbooks updated to interpret per-hop metrics, path-change
  events, and protocol-fallback behavior; troubleshooting guide
  published before GA.
- *Internal Adoption*: Dynatrace internal NetOps/SRE adopts the
  expanded traceroute monitors before customer-facing rollout.
- *Licensing*: Treated as additive enhancement under the existing NAM
  synthetic entitlement — no new SKU or pricing change planned.
  Product Marketing to confirm packaging language before GA.
- *Marketing*: Release note and short demo video for the launch;
  competitive-positioning blog post; field enablement deck for AEs
  and SEs including the parity matrix.
- *Sales / Field*: SE enablement covering when to use which probe
  protocol, how to interpret path-change events, and head-to-head
  positioning against Datadog Network Path and SolarWinds NetPath.


## E2E Acceptance Criteria

1. *Probe protocol selection*: NAM Traceroute monitors accept ICMP,
   UDP, or TCP as the probe protocol, with TCP as the recommended
   default for paths that block ICMP; misconfiguration is surfaced
   with a clear validation message.
2. *Protocol fallback*: When the configured probe protocol fails to
   resolve a hop, the runner can optionally fall back to a secondary
   protocol per a documented policy; behavior is logged on the result.
3. *Execution output*: Each Traceroute execution returns an ordered
   hop list with per-hop latency, packet loss, and jitter, with
   explicit sample size, up to a configurable max-hops limit.
4. *Configurable execution*: Operators can configure max-hops,
   per-hop probe count, per-hop timeout, and path-change sensitivity
   with safe defaults.
5. *Hop enrichment*: Every resolved hop in a Traceroute result is
   enriched with reverse DNS (when available) and ASN/owner lookup,
   and the result includes layer segmentation (LAN / WAN / ISP /
   cloud / target zone) where determinable.
6. *Path-change detection*: Path changes (hop sequence, ASN sequence,
   or layer transitions) are detected against a rolling baseline and
   emitted as discrete events queryable and alertable end-to-end.
7. *Historical path diff*: Operators can open any Traceroute execution
   and see a side-by-side diff against the previous execution and
   against the last known good path, with changed hops visually
   marked.
9. *Alerting on path and hops*: Alerts can target path-change events
   and per-hop loss and latency thresholds, with deterministic
   evaluation and sample-size guards under low sample counts.
10. *Persistence and query*: Hop sequences and per-hop time series are
    persisted and queryable in existing dashboarding and historical
    analysis surfaces end-to-end.
11. *Multi-source probing*: Traceroute monitors can run from multiple
    NAM sources to the same target, and results can be compared
    across sources in the result view.
12. *Documentation*: Customer documentation covers configuration,
    metric interpretation, recommended thresholds, migration guidance,
    and a competitive positioning section with the parity matrix.
14. *Competitive parity (per Competitive Landscape)*: Side-by-side
    review against Datadog Network Path and SolarWinds NetPath for
    the three reference scenarios (long-haul WAN, ICMP-blocked path,
    multi-cloud egress) shows no missing capability from the parity
    matrix items committed to this VI.

## Customer Zero Planning

- *Subset for Customer Zero*: Traceroute multi-protocol (ICMP / UDP /
  TCP probes) with automatic fallback, per-hop enrichment, path-change
  detection, and the path-diff view. Alerting on path-change events
  and per-hop loss/latency.
- *Adoption plan (proposed)*:
  - *Sprint S+1*: Internal NetOps converts ~20  representative
    extension traceroute monitors across at least 3 production regions to the
    new multi-protocol runner with default profiles only — no custom
    values yet.
  - *Sprint S+2*: Expand to ~100 monitors with custom profiles for
    three known problematic paths (long-haul WAN, ICMP-blocked
    firewalled segment, multi-cloud egress); enable alerting on
    path-change events and per-hop loss/latency.
  - *Sprint S+3*: Onboard the design-partner customer to Private
    Preview on ~15 of their existing traceroute monitors with no
    mandatory migration. 
- *Success signal*: Customer Zero resolves at least one representative
  routing or peering incident using the new metrics without falling
  back to external tooling, reports zero critical regressions on the
  converted monitors, and confirms alert volume stays within ±10 %
  of the pre-conversion baseline.
- *Exit from Customer Zero*: Internal adoption hits 100 % of planned
  monitors with success signal met, design-partner sign-off recorded,
  and no open Sev-1 / Sev-2 issues against the new code paths.

## E2E Demo (for acceptance)

1. Open monitor creation, choose Traceroute, select TCP as the probe
   protocol, set max hops and per-hop probe count, save.
2. Run the monitor against a target reachable only through an
   ICMP-blocking firewall; show hop list with reverse DNS, ASN/owner,
   layer segmentation, and per-hop latency/loss/jitter.
3. Re-run with the probe protocol set to ICMP against the same target;
   show the runner fall back to TCP per the configured policy and
   note this in the result view.
4. Re-run the monitor after forcing a path change (for example switch
   source region) and show the path-diff view side-by-side with the
   previous path; show the path-change event in the events surface.
5. Open an alert template, set thresholds on path-change events,
   per-hop loss, and per-hop latency; demonstrate sample-size guarded
   evaluation.
6. Open a dashboard or query surface and show per-hop metrics and
   path-change events trending over time.
7. Run the head-to-head competitive demo: same target, same time,
   NAM Traceroute alongside Datadog Network Path and SolarWinds
   NetPath; walk through the parity-matrix items and show no missing
   capability for the items committed to this VI.
8. Show the published help page covering the new options and the
   competitive positioning section.

## Out of Scope


- NetFlow enrichment and flow-based diagnostics.
- L7 transaction synthetic scripting enhancements.
- Cloud topology overlay (AWS / Azure / GCP region, VPC, and managed
  service labels on traceroute hops) — deferred to a sibling VI..
- Full packet-capture-based diagnostics.
- Synthetic App UI redesign beyond the minimal additions required to expose new
  fields, the hop-by-hop visualization, the path-diff view, and the
  competitive-positioning help content.

## Success Metrics

Evaluated 3–6 months post-GA.

- *Diagnostic self-sufficiency*: ≥ 70 % of traceroute-related support
  escalations resolved using NAM data only, without falling back to
  external tooling (baseline: current escalation rate from Support
  ticket tags).
- *Adoption — new monitors*: ≥ 60 % of the traceroute monitors
  created in the first 3 months post-GA use at least one expanded
  option (non-ICMP probe protocol, sample-size-aware threshold, or
  path-change alerting).
- *Adoption — fleet migration*: ≥ 150 existing traceroute monitors
  migrated to the expanded configuration within 6 months post-GA
  across internal Dynatrace + design-partner cohort.
- *MTTR*: ≥ 30 % reduction in median time-to-diagnose for path-level
  network incidents vs. the pre-GA baseline.
- *Alert quality*: ≥ 50 % reduction in false-positive rate on
  per-hop and path-change alerts after enabling sample-size-aware
  and baseline-aware thresholds (compared to the same monitors before
  migration).
- *Path-change signal value*: ≥ 1 unplanned routing or peering issue
  per Customer Zero per quarter is detected by path-change alerts
  *before* end-user impact is reported, demonstrating proactive value.
- *Competitive — head-to-head*: Dynatrace can check the "traceroute"  box in all RFP,RFI
- *Competitive — gap report*: At GA, no item that the parity matrix
  marks `yes` for both Datadog and SolarWinds is missing from NAM
  Traceroute, excluding the explicit deferrals in *Out of Scope*.

## Cost Analysis
To be decided later. A few proposals:
- Option 1:
  metrics based: similar metric numbers as ICMP or TCP NAM:  target based.
- Option 2:
  metrics based: similar metric numbers as ICMP or TCP NAM:  hop based.


## Cost Analysis

- *Engineering effort (proposed t-shirt: L)*: Five parallel workstreams
  at ~2 sprints each, plus ~1 sprint for documentation and field
  enablement. Refine after Dev Epic breakdown.
  - NAM Traceroute multi-protocol runner (ICMP / UDP / TCP) with
    automatic fallback.
  - Hop-enrichment service (reverse DNS + ASN/owner lookup with
    caching).
  - Path-comparison service (diffing, last-known-good baseline,
    path-change event emission).
  - Monitor schema, API extensions, and data pipeline + storage
    changes for hop sequences and per-hop time series.
  - Configuration UI + result view (hop visualization, path-diff,
    competitive-positioning content).
- *Storage and ingest*: Per-run footprint grows materially because hop
  sequences and per-hop time series are stored. Provisional estimate:
  expanded traceroute records ~5–10× the per-row size of the current
  legacy traceroute output, depending on average path length. Confirm
  with data platform; expect a dedicated retention / aggregation
  policy for per-hop history (for example raw for N days, daily
  aggregates beyond).
- *Compute*: Hop enrichment adds reverse-DNS and ASN-lookup cost —
  design must cache aggressively to keep per-execution cost low.
  Path-comparison is cheap per execution but accumulates with monitor
  count. Validate under scale testing before GA.
- *External lookups*: ASN/owner lookup needs a maintained data source
  (internal mirror of routing data or a vetted external feed).
  Confirm source, refresh cadence, and any licensing cost.
- *Operational*: Moderate support load increase during preview as
  customers adopt path-change alerts (tuning takes time). Offset
  post-GA by reduced escalation volume on traceroute diagnostics.


## Section-by-section guidance applied

- *WHY* uses the "Scenario Breakdown" pattern (where it hurts) since
  the pain is concrete and multi-faceted, extended with competitor
  evidence to motivate scope.
- *Competitive Landscape* is an added section (not in the standard
  VCG template) used here as supporting context for WHY and as the
  source of the parity acceptance criteria; the parity matrix gives
  the field team a tool they can quote in deals.
- *Acceptance Criteria* uses the "Grouped by feature/category" pattern,
  with bold labels per criterion area (protocol, execution,
  enrichment, path change, alerting, compatibility, competitive).
- Section names and order follow the VCG 2023 official template; the
  *Competitive Landscape* section sits between WHY and Functional
  Requirements where it provides the most context. This file is in
  Markdown for local editing — convert headings to `h2.` / `h3.` Jira
  wiki markup and the parity-matrix table to Jira table markup when
  copying into the Jira VI.

## Completeness checklist — target state: Problem Stated

- [x] Short Abstract / Blogline
- [x] Target Audience
- [x] WHY with concrete pain points
- [x] Customer Zero identified (proposed assignment recorded)
- [x] Functional Requirements / Solution Journey
- [x] Non-Functional Requirements
- [ ] Target end date / Fix version (set in Jira)
- [ ] Assignee PM/PA (set in Jira)
- [x] Out of Scope drafted
- [x] Effort estimate / t-shirt sizing (proposed L — confirm at Dev
      Epic breakdown)
- [x] Enablement Requirements outlined
- [x] Cost Analysis outlined

## Completeness checklist — target state: Use Case Defined

- [x] E2E Acceptance Criteria
- [x] Customer Zero Planning (sprint-by-sprint plan and exit criteria)
- [x] E2E Demo
- [x] Out of Scope
- [x] Success Metrics with proposed targets
- [x] Cost Analysis with proposed effort estimate
- [x] Dependencies identified (informational list in Solution Journey;
      link the corresponding Jira items)
- [ ] PE accepts VI (Jira action)
- [ ] High-fidelity mockups for new configuration form and result view
      (UX deliverable — track in design ticket)
- [ ] Architectural concept document (PA deliverable — link from Jira)
- [ ] Design QA sign-off (UX-heavy parts only — applies to the form
  and result view additions)
