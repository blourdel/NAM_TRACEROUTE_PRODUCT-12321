# VI PRODUCT-12321 — NAM Traceroute

## Short Abstract / Blogline

NAM Traceroute synthetic test with multi-protocol probes
(ICMP, UDP, TCP), per-hop enrichment, path-change detection, and
historical path diff, so operators can resolve path-level network
issues using NAM data alone — at parity with Datadog Network Path and
SolarWinds NetPath.

It will be used by the upcoming network app for a the "network path" feature.

## Customer Zero

- Internal: Dynatrace internal EDE/SRE team operating production
  NAM Traceroute monitors across multi-region infrastructure. They need
  hop-visibility, path-change, and ICMP-blocked-path. Customer Zero validates: traceroute multi-protocol behavior, per-hop
enrichment accuracy, path-change event quality, alert quality under
low sample counts.

- External:  A number of customers are already using the traceroute extension. They will be approached to be design partners of NAM traceroute.

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

## WHY?

- MTTR does not decline when using Dynatrace, deals are lost on head-to-head
capability questions, and customers question whether NAM is sufficient
for network synthetic monitoring.

- Ping or TCP NAM  validates that a target is reachable along *some*
path but stops short of the diagnostic depth operators need for modern
network troubleshooting. As a result, investigations escalate to
external tools (mtr, traceroute, tcptraceroute, packet captures) or
to competitor products  (SolarWinds NetPath, etc.) that
already provide multi-protocol probes, per-hop enrichment, path-change
detection, and alerting.

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

External tooling and competitor products fill these gaps today, which
fragments the operator workflow and weakens the case for NAM as the
primary network synthetic platform.

## Competitive Landscape

Datadog and SolarWinds set the de-facto bar for traceroute and
path-analysis capability. NAM must reach parity to remain credible in
head-to-head deals and to remove the operator workflows that currently
bridge to external tools for hop-level diagnostics.

### Datadog — Network Performance Monitoring and Network Path

- Hop-by-hop path analysis between source agent and destination
  endpoint, run continuously on a schedule.
- Multi-protocol probes (ICMP, UDP, TCP) with automatic fallback so
  paths resolve even when intermediate devices rate-limit or drop ICMP.
- Per-hop RTT, packet loss, and jitter with sample-size context.
- Reverse DNS and ASN/owner enrichment on every hop.
- Cloud topology overlay — hops on AWS, Azure, and GCP are labelled
  with region, VPC, and managed-service identifiers.
- Path-change detection with historical diff and replay.
- Native alerting on path change, per-hop RTT, and per-hop loss.
- Tight integration with Datadog APM, Synthetics, and Service Map for
  end-to-end correlation.

  [Datadog traceroute blog](https://www.datadoghq.com/blog/network-test-protocols/)
  [Datadog traceroute documentation](https://docs.datadoghq.com/synthetics/network_path_tests/)

### SolarWinds — NetPath, NPM, Engineer's Toolset

- TCP-based path discovery as the *default* mode (NetPath), because
  ICMP is dropped or rate-limited on many enterprise and ISP paths;
  UDP and ICMP modes available as alternatives.
- Hop-by-hop RTT, packet loss, and jitter per probe interval.
- ASN and owner identification per hop, plus network-layer segmentation
  (LAN, WAN/ISP, cloud, target-service zone) shown visually.
- Continuous probing from multiple polling agents to the same target,
  with cross-agent comparison.
- Path-change tracking with historical replay and snapshot diff.
- Alerting on path change, hop RTT, and hop loss; integrated into
  NPM and Server & Application Monitor.
- Engineer's Toolset adds ad-hoc traceroute and related utilities for
  on-demand investigation outside of scheduled monitoring.

### Parity Matrix

`NAM target this VI` = parity items committed to in this VI's scope.
`future VI` = explicitly deferred to a sibling VI to keep this one
shippable. `partial` = exists today but does not meet operator
expectations versus the competitive bar.

| Capability                                  | Datadog   | SolarWinds   | Dynatrace today | DT after this VI   |
|---------------------------------------------|:---------:|:------------:|:---------------:|:------------------:|
| ICMP-probe traceroute                       | yes       | yes          | Extension       | yes                |
| TCP-probe traceroute                        | yes       | yes          | no              | yes                |
| UDP-probe traceroute                        | yes       | yes          | no              | yes                |
| Per-hop RTT                                 | yes       | yes          | Extension       | yes                |
| Per-hop jitter                              | yes       | yes          | no              | yes                |
| Reverse DNS enrichment per hop              | yes       | yes          | no              | yes                |
| ASN / owner enrichment per hop              | yes       | yes          | no              | No                 |
| Configurable max-hops and probe count       | yes       | yes          | Extension       | yes                |
| Path-change detection                       | yes       | yes          | No              | No                 |
| Historical path diff / replay               | yes       | yes          | no              | No                 |
| Native alerting on path change              | yes       | yes          | no              | No                 |
| Native alerting on per-hop loss / RTT       | yes       | yes          | no              | yes                |
| Ad-hoc on-demand probe                      | partial   | yes          | no              | yes                |
| Per-hop response %                          | No        | No           | No              | Yes                |

### Priority implications for this VI

- *Probe-protocol parity is non-negotiable*: TCP-probe traceroute is
  the default in SolarWinds NetPath because ICMP may be rate-limited or
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
   (new hop sequence, new ASN appears, or per-hop RTT/loss crosses
   a threshold) and opens the historical path diff to see exactly
   which hops changed and when.
3. Platform admin rolls out the expanded traceroute capability with
   safe defaults so existing traceroute monitors continue to run
   unchanged, then switches selected monitors to TCP-probe mode where
   ICMP is blocked.
4. SRE sets thresholds on per-hop loss and per-hop RTT with
   deterministic behavior under low sample counts (for example
   "evaluate only when sample size ≥ N").
5. Investigator opens a historical traceroute execution and side-by-side
   diffs it against the previous execution and the last known good
   path to confirm a routing change as root cause of an incident.
6. Analyst queries historical NAM data to trend per-hop metrics and
   path-change frequency across regions and correlate with incidents.
7. During traceroute filtering Traceroute Hops and destination can be selected based on properties defined by the user.

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
  whose network the RTT is in."
- *Platform admin*: "As a platform admin, I want safe defaults for
  the new options, so my existing traceroute monitor fleet keeps
  running without a forced migration."
- *Alerting owner*: "As an alerting owner, I want sample-size-aware
  thresholds, so one hop blip doesn't page on-call."
- *Analyst*: "As an analyst, I want per-hop metrics and path-change
  events available in DQL and dashboards, so I can correlate regional
  path trends with incidents."

### Prior work

- [Traceroute extension](https://docs.dynatrace.com/docs/observe/infrastructure-observability/extensions/traceroute)
- [Traceroute Hub](https://www.dynatrace.com/hub/detail/traceroute/?query=traceroute)
- [Dev environment](https://yts0540d.dev.apps.dynatracelabs.com/ui/apps/my.traceroute.map/)
- traceroute_extension.yaml (see attached)

#### Scope

- ICMP only on Linux and Windows
- Traceroute entity linked to a network:device entity

### Solution Concept

- Introduce NAM Traceroute, single target, multi-protocol implementation supporting ICMP, UDP, and TCP probes with automatic fallback when a protocol is rate-limited or blocked.
- Extend monitor schema and API to expose new options — probe rotocol, max-hops, per-hop probe count, per-hop timeout, per-hop limit.
- Add a hop  IP enrichment service that resolves reverse DNS and looks up ASN/owner.
- Add a path-comparison screen that diffs each new execution against a rolling "last known good" path select from the last 24 hours maximum and emits a path-change event when hop sequence, ASN sequence, or layer segmentation changes.
- Surface new fields in the monitor configuration UI including a tabular form, single traceroute hop-by-hop visualization, a path-diff view. 
- Provide OOTB box alerting with default values.

### Configuration

- Frequency
- number of traceroutes to execute

#### Performance

Future support for "Spaces" is expected to enable per monitor types default values.

Per target:

- RTT, recommended value
- Jitter, recommended value
- Max Hop, recommended value

#### Frequency

As per other NAM: 1 minute minimum, to 1 hour and custom

#### Alerting

##### Health Alerts

- Per request performance
  - RTT
  - Jitter
  - Per hop delay

- Failed monitor

- Perhop to hop delay

##### Warning Signals

- Based on user setting: Target not reached
- Based on user setting: Per hop threshold or baseline
- Path variation: configurable based on the number of hop variation

##### INFO Signals

- Path variation: configurable based on the number of hop variation

#### Outage and performance

-Default OFF: Generate a problem and send an alert when the monitor is unavailable at all configured locations (global outage).

- Default ON: Generate a problem and send an alert when the monitor is unavailable for one or more consecutive runs at any location.

### Metrics and dimensions

#### Dimensions

All NAM dimensions complemented by:

- Hop id

#### Metrics

Per target:

- Jitter
- RTT 
- Packet loss

Per hop:

- Jitter
- RTT
- Packet loss
- Traversal count (number of traceroute going through this hop)

Note:Metrics are reported on a per hop basis, When a target fails, no other metric is  written, The performance metrics are written in all cases

### Delivery Phases based on market priority

The phases content may be altered based on engineering input, as it may be more efficient to group some feature for efficiency purposes.

IPv4, IPv6 are supported for all features.

#### December 26 Rally

- Linux traceroute runner, including containerized, private locations only : ICMP, TCP SYN and TCP SACK
- Reverse DNS enrichement
- Hop-by-Hop jitter, RTT, packet loss metrics (per hop and target)
- Max hop and probe count
- Minimal individual path visualization
- Alerting
- On demand probes (usual 2-3 minute delay)

### Dependencies

- *Traceroute runner*: Windows "tracert" only supports ICMP, while Linux "traceroute" supports ICMP, UDP and TCP SYN
- *Hop-enrichment service*: reverse DNS resolver and ASN/owner lookup
  pipeline with caching. Confirm whether an existing internal service
  can be reused or a new lightweight component is needed.
- *Path-comparison service*: diffing engine and path-change event
  emission. Likely a new service co-located with the NAM result
  pipeline.
- *UI/UX*: Graphical component.
- *Alerting platform*: support for path-change events and per-hop
  thresholds in the existing alerting surface.

## Non-Functional Requirements

- *Platform*: This is supported in Gen3 only. No support of Gen2.
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
  Product Marketing to confirm packaging language before GA.
- *Marketing*: Release note and short demo video for the launch;
  competitive-positioning blog post; field enablement deck for AEs
  and SEs including the parity matrix.
- *Sales / Field*: SE enablement covering when to use which probe
  protocol, how to interpret path-change events, and head-to-head
  positioning against Datadog Network Path and SolarWinds NetPath.

## December 26 Rally E2E Acceptance Criteria

- Probe protocol selection: NAM Traceroute monitors accept ICMP,
   UDP, or TCP as the probe protocol, with TCP as the recommended
   default for paths that block ICMP.
- *Execution output*: Each Traceroute execution returns an ordered
   hop list with per-hop RTT, packet loss, and jitter, with
   explicit sample size, up to a configurable max-hops limit.
- *Configurable execution*: Operators can configure max-hops,
   per-hop probe count, per-hop timeout sensitivity
   with safe defaults.
- *Hop enrichment*: Every resolved hop in a Traceroute result is
   enriched with reverse DNS (when available).
- Alerting on target and hops: Alerts can target per-hop loss, jitter and RTT thresholds.
- Persistence and query: Hop sequences and per-hop time series are
    persisted and queryable in existing dashboarding and historical
    analysis surfaces end-to-end.
- Multi-location probing: Traceroute monitors can run from multiple
    location to the same target but to a single target.

## December 26 Rally Customer Zero Planning

- *Subset for Customer Zero*: Traceroute multi-protocol (ICMP / UDP /
  TCP probes) with automatic fallback, per-hop enrichment, path-change
  detection. Alerting on per-hop loss/RTT.
- *Success signal*: Customer Zero resolves at least one representative
  routing or peering incident using NAM traceroute without falling
  back to external tooling.
- *Exit from Customer Zero*: Internal adoption hits 100 % of planned
  monitors with success signal met, design-partner sign-off recorded,
  and no open Sev-1 / Sev-2 issues against the new code.

## E2E Demo (for acceptance)

1. Open monitor creation, choose Traceroute, select TCP as the probe
   protocol, set max hops and per-hop probe count, save.
2. Run the monitor against a target reachable only through an
   ICMP-blocking firewall; show hop list with reverse DNS, ASN/owner,
   layer segmentation, and per-hop RTT/loss/jitter.
3. Open an alert template, set thresholds on path-change events,
   per-hop loss, and per-hop RTT.
4. Open a dashboard or query surface and show per-hop metrics and
   path-change events trending over time.
5. Run the head-to-head competitive demo: same target, same time,
   NAM Traceroute alongside Datadog Network Path and SolarWinds
   NetPath; walk through the parity-matrix items and show no missing
   capability for the items committed to this VI.
6. Show the published help page covering the new options and the
   competitive positioning section.

## Out of Scope

- Windows: ICMP, TCP SYN and TCP SACK
- Autonomous System (BGP) enrichement
- NetFlow enrichment and flow-based diagnostics.
- Application traffic enrichment.
- Single traceroute visual representation
- Alterting baseline (if no generic solution is available at implementation time)
- Public locations
- Multi target NAM.

## Success Metrics

Evaluated 3–6 months post-GA.

- *Diagnostic self-sufficiency*: ≥ 70 % of traceroute-related support
  escalations resolved using NAM data only, without falling back to
  external tooling (baseline: current escalation rate from Support
  ticket tags).
- *Adoption — new monitors*: ≥ 60 % of the traceroute monitors
  created in the first 3 months post-GA use at least one expanded
  option (non-ICMP probe protocol,
  path-change alerting).
- *Path-change signal value*: ≥ 1 unplanned routing or peering issue
  per Customer Zero per quarter is detected by path-change alerts
  *before* end-user impact is reported, demonstrating proactive value.
- *Competitive — head-to-head*: Dynatrace can check the "traceroute"  box in all RFP,RFI
- *Competitive — gap report*: At GA, no item that the parity matrix
  marks `yes` for both Datadog and SolarWinds is missing from NAM
  Traceroute, excluding the explicit deferrals in *Out of Scope*.

## Pricing

It follows the  NAM synthetic entitlement (metric based).
The billed metrics are defined in the "metrics and dimensions" section.

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
protocol-specific reply (see below). RTT for each hop is the
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
    *inflated per-hop RTT* that does not reflect the RTT a
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
| Diagnosing RTT a real flow would see           | TCP (matches production hashing best)   |
