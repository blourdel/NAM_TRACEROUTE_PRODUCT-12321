# Traceroute Extension — Suggested Metrics

This document defines the NAM traceoute metrics.



### Path-level Metrics (end-to-end)

These align with existing `traceroute.*` keys and extend coverage to match NAM ICMP equivalents.

| Metric Key | Display Name | Unit | NAM Analog | Description |
|---|---|---|---|---|
| `traceroute.path.hops_count` | Traceroute Hop Count | Count | — | Total number of hops from source to destination |
| `traceroute.path.loss_rate` | Traceroute Path Loss Rate | Percent | `icmp.success_rate` | Ratio of lost probes to sent probes, end-to-end |
| `traceroute.path.packets_sent` | Traceroute Packets Sent | Count | `icmp.packets_sent` | Total probes sent during the traceroute execution |
| `traceroute.path.packets_received` | Traceroute Packets Received | Count | `icmp.packets_received` | Total probes that received a response |
| `traceroute.path.execution_time` | Traceroute Execution Time | MilliSecond | `icmp.request_execution_time` | Total wall-clock time of the full traceroute run |

Suggested additional dimension: `request.type` (values: `icmp`, `udp`, `tcp`) to align with NAM's `request.type` dimension and support multi-protocol comparisons.

---

### Hop-level Metrics (per intermediate node)

These enable per-hop analysis of the network path, directly inspired by NAM's per-request dimensions.

| Metric Key | Display Name | Unit | NAM Analog | Description |
|---|---|---|---|---|
| `traceroute.hop.rtt` | Traceroute Hop RTT | MilliSecond | `icmp.round_trip_time` | RTT measured at a specific hop in the path |
| `traceroute.hop.loss_rate` | Traceroute Hop Loss Rate | Percent | `icmp.success_rate` | Ratio of lost probes to sent probes at a specific hop |
| `traceroute.hop.packets_sent` | Traceroute Hop Packets Sent | Count | `icmp.packets_sent` | Number of probes sent to this hop |
| `traceroute.hop.packets_received` | Traceroute Hop Packets Received | Count | `icmp.packets_received` | Number of responses received from this hop |
| `traceroute.hop.reachable` | Traceroute Hop Reachable | Count | — | 1 = hop responded, 0 = hop timed out (`*`) |

Suggested dimensions for hop-level metrics:

| Dimension Key | Description | NAM Analog |
|---|---|---|
| `hop.index` | Position of the hop in the path (1-based TTL) | `step.id` |
| `hop.address` | IP address of the intermediate node | `request.target.address` |
| `hop.name` | DNS name of the intermediate node (if resolved) | — |
| `device.name` | Destination device name (existing) | — |
| `device.address` | Destination device address (existing) | `request.target.address` |
| `dt.security_context` | Security context (existing) | `dt.security_context` |

---

### TCP-specific Metrics

For traceroutes run in TCP mode (if the extension supports or will support it), following `dt.synthetic.multi_protocol.tcp.*`:

| Metric Key | Display Name | Unit | NAM Analog | Description |
|---|---|---|---|---|
| `traceroute.tcp.connection_time` | Traceroute TCP Connection Time | MilliSecond | `tcp.connection_time` | Time to complete a TCP handshake at the destination |
| `traceroute.tcp.connection` | Traceroute TCP Connected | Count | — | 1 = TCP handshake succeeded, 0 = failed |

Suggested additional dimension: `request.tcp_port_number` (aligns with NAM TCP dimension).

---

### UDP-specific Metrics

UDP is connectionless: there is no handshake, so there is no equivalent to `tcp.connection_time`. Instead, UDP traceroute works by sending probes with increasing TTL and detecting destination arrival via an ICMP **Port Unreachable** response (as opposed to ICMP **Time Exceeded** for intermediate hops). Metrics therefore measure probe round-trip behavior and ICMP-signaled reachability rather than connection state.

| Metric Key | Display Name | Unit | TCP Analog | Description |
|---|---|---|---|---|
| `traceroute.udp.probe_rtt` | Traceroute UDP Probe RTT | MilliSecond | `traceroute.tcp.connection_time` | Time from sending a UDP probe to receiving the ICMP response (Time Exceeded or Port Unreachable). No handshake equivalent — this is the only latency signal available over UDP. |
| `traceroute.udp.destination_reached` | Traceroute UDP Destination Reached | Count | `traceroute.tcp.connection` | 1 = ICMP Port Unreachable received from destination (probe arrived), 0 = timeout. Named "reached" rather than "connected" to reflect the connectionless nature of UDP. |
| `traceroute.udp.response_rate` | Traceroute UDP Response Rate | Percent | — | Ratio of ICMP responses received to UDP probes sent. Captures overall probe responsiveness where no per-packet ACK exists. |
| `traceroute.udp.packets_sent` | Traceroute UDP Packets Sent | Count | — | Total UDP probes sent during the traceroute execution. |
| `traceroute.udp.packets_received` | Traceroute UDP Packets Received | Count | — | Total ICMP responses received (Time Exceeded + Port Unreachable). |

Suggested additional dimension: `request.udp_port_number` (mirrors `request.tcp_port_number`; UDP traceroute targets a high port such as 33434+ by default).

---

### Jitter Metrics

Jitter measures the variability of RTT across multiple probes to the same target. It is computed as the **mean absolute difference between consecutive RTT samples** (RFC 3550 definition): `jitter[n] = |RTT[n] - RTT[n-1]|`, averaged over all probe pairs in a run.

Jitter applies at every level where RTT is already measured — path, hop, and per-protocol. There is no NAM analog (NAM does not expose jitter directly), so these are traceroute-native additions.

#### Path-level jitter

| Metric Key | Display Name | Unit | RTT Source | Description |
| --- | --- | --- | --- | --- |
| `traceroute.path.jitter` | Traceroute Path Jitter | MilliSecond | `traceroute.rtt` | Jitter of end-to-end RTT samples across consecutive probes to the destination |

#### Hop-level jitter

| Metric Key | Display Name | Unit | RTT Source | Description |
| --- | --- | --- | --- | --- |
| `traceroute.hop.jitter` | Traceroute Hop Jitter | MilliSecond | `traceroute.hop.rtt` | Jitter of RTT samples at a specific intermediate hop across consecutive probes. Shares the same `hop.index`, `hop.address`, `hop.name` dimensions as `traceroute.hop.rtt`. |

#### Protocol-specific jitter

| Metric Key | Display Name | Unit | RTT Source | Description |
| --- | --- | --- | --- | --- |
| `traceroute.icmp.jitter` | Traceroute ICMP Jitter | MilliSecond | `traceroute.rtt` (ICMP mode) | Jitter of ICMP probe RTTs. Equivalent to path-level jitter when `request.type=icmp`; exposed separately for protocol-isolated dashboards. |
| `traceroute.tcp.connection_jitter` | Traceroute TCP Connection Jitter | MilliSecond | `traceroute.tcp.connection_time` | Jitter of TCP handshake durations across consecutive probes. Measures instability in connection setup time, not data transfer. |
| `traceroute.udp.probe_jitter` | Traceroute UDP Probe Jitter | MilliSecond | `traceroute.udp.probe_rtt` | Jitter of UDP probe RTTs (time between sending probe and receiving ICMP response). Because UDP is connectionless, jitter here captures ICMP response variability, not connection-state changes. |

---

## Summary: Mapping to NAM Convention

| NAM ICMP Metric | Traceroute Equivalent (path) | Traceroute Equivalent (hop) |
|---|---|---|
| `icmp.round_trip_time` | `traceroute.rtt` *(existing)* | `traceroute.hop.rtt` |
| `icmp.success_rate` | `traceroute.path.loss_rate` | `traceroute.hop.loss_rate` |
| `icmp.packets_sent` | `traceroute.path.packets_sent` | `traceroute.hop.packets_sent` |
| `icmp.packets_received` | `traceroute.path.packets_received` | `traceroute.hop.packets_received` |
| `icmp.request_execution_time` | `traceroute.path.execution_time` | — |
| `tcp.connection_time` | `traceroute.tcp.connection_time` | `traceroute.udp.probe_rtt` *(connectionless analog)* |
| `tcp.connection` *(implicit)* | `traceroute.tcp.connection` | `traceroute.udp.destination_reached` |
| — *(no NAM analog)* | `traceroute.path.jitter` / `traceroute.icmp.jitter` | `traceroute.hop.jitter` |
| — *(no NAM analog)* | `traceroute.tcp.connection_jitter` | `traceroute.udp.probe_jitter` |

---

## Notes

- All suggested keys use the `traceroute.*` prefix to remain consistent with the existing extension namespace rather than adopting the `dt.synthetic.*` prefix, which is reserved for first-party Dynatrace synthetic monitors.
- Hop-level metrics will significantly increase cardinality. Consider a feature flag or activation schema option to enable/disable them.
- The commented-out `traceroute.loss_percent` key in `extension.yaml` maps to `traceroute.path.loss_rate` above (with a rename to align with the NAM `*_rate` suffix convention).
- UDP has no `connection_time` equivalent because there is no three-way handshake. `traceroute.udp.probe_rtt` is the closest analog: it measures the time to receive an ICMP response to a UDP probe, which is the only round-trip signal UDP can produce. Similarly, `traceroute.udp.destination_reached` is preferred over "connected" to avoid implying a stateful connection.
- Jitter requires at least two consecutive probe RTT samples within a single run. If only one probe is sent, the jitter value should be omitted rather than reported as 0 to avoid misleading baselines.
