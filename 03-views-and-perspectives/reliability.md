# Reliability perspective — connectivity failure and degradation

How the system behaves under connectivity/cloud failures. Principle: **the safety-critical never depends fully on the cloud**, and **the delay-tolerant is always buffered and catches up**. Decisions are in [ADR-017](../04-adrs/ADR-017-connectivity-failure-degraded-operation.md), channels in [ADR-004](../04-adrs/ADR-004-edge-connectivity-mesh-cellular-data-mule.md), buffering in [ADR-003](../04-adrs/ADR-003-edge-mqtt-store-and-forward.md).

## Data classes (reminder)
- **Real-time-critical:** a venomous animal escaped/got sick, fire, missing-child search, payment → fixed/cellular channel, **backup mandatory**.
- **Delay-tolerant:** routine telemetry, queue metrics, drive-by snapshots, 360 → edge buffer, catches up over Wi-Fi/cellular; **data mule — from R3** (shuttle, reduces the cost of connectivity for remote enclosures).

## Degradation matrix

> The matrix describes the **target state**. In the **MVP** (without the Q10 shuttle) resilience rests on **edge store-and-forward + Wi-Fi/mesh/radio bridges (LoS) + cellular (dual-carrier) + local safety alerts**; the **data mule and shuttle autonomy are R3** (a reinforcement, not a condition of the MVP). Phases — [mvp-vs-roadmap](../02-solution/mvp-vs-roadmap.md).

| Failure scenario | Works | Degrades | Unavailable |
|---|---|---|---|
| **Wi-Fi outage in a zone** (patchy) | ticket entry (offline cache Q2), edge-CV locally, telemetry buffering (+shuttle autonomy — from R3) | analytics/heatmap freshness (data will arrive in a batch), route quest (on cache) | — (nothing is lost, catches up) |
| **Failure of one cellular carrier** | everything — critical alerts go over the **second carrier/satellite** (dual-carrier) | throughput of the critical channel | — (given a backup) |
| **Full/prolonged loss of the channel to the cloud** | ticket entry (offline), **local safety alerts** at critical enclosures (siren/LAN/mesh), edge-CV, buffering (+shuttle autonomy fail-safe — from R3) | ticket sales (offline queue with limits / fail-closed), NL analytics, social marketing (Q11), cloud AI inference, schedules (Q12) | real-time cloud analytics, model update/rollout, Q11 publications |
| **Cloud zone (AZ) failure** | everything (MVP — multi-AZ in a single region) | — | multi-region — roadmap |

## Key guarantees
- **Safety-first:** a threshold alert at a critical enclosure fires locally, even if the cloud/carrier is unavailable (a deterministic edge rule, not AI).
- **No single-carrier SPOF** for the critical class (dual-carrier + optional satellite).
- **Sales:** validation of purchased tickets — always offline; new sales — an offline queue with idempotency and limits, worst-case mode fail-closed.
- **Data is not lost:** the delay-tolerant is buffered (store-and-forward), idempotent upload on recovery; **in the MVP** it catches up over Wi-Fi/cellular, **from R3** the data mule reduces the cost of delivery from remote enclosures.

## Target indicators (assumption, to be calibrated)
- Availability of safety alerts at critical enclosures: does not depend on the cloud (local path).
- SLA of business functions: 3–4 nines (a deliberate trade-off for cost).
- RPO of delay-tolerant data: bounded by the size of the edge buffer (hours–days); RTO of sales on an outage: recovery of the payment queue after connectivity returns.
