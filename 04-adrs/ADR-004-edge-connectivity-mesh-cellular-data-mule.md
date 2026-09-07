# ADR-004: Edge connectivity — hybrid mesh / cellular / data-mule with an area threshold

## Status
Accepted

## Context
Data must not only be buffered (see [ADR-003](ADR-003-edge-mqtt-store-and-forward.md)) but also physically delivered under patchy Wi-Fi (brief fact, §E) from heterogeneous, spread-out territory ("large and sprawling", brief fact, §A). Running fixed Wi-Fi/wiring to all 55 enclosures (brief fact, §C) is expensive; placing a cellular gateway on each is linearly expensive. Some data is not latency-critical, some (an alert about an escaped/sick venomous animal) is critical. Area and break-even estimate (assumption; operational footprint ≈100 acres, remote enclosures ≈5-40 of 55): the break-even threshold "gateways vs data mule" is **N\* ≈ 2-6 remote enclosures** (details — `../03-views-and-perspectives/deployment.md`, `../03-views-and-perspectives/cost.md`).

## Decision
Not a "silver bullet" but a **hybrid whose threshold is set by area**: (1) fixed **Wi-Fi/mesh** where it is cost-effective; (2) **shuttle-mesh** along routes; (3) **data mule** (the shuttle picks up the buffer as it passes) for remote delay-tolerant enclosures when N exceeds the threshold (shuttle — roadmap R3); (4) **cellular gateway/uplink** — always for real-time-critical enclosures, regardless of N; (5) **directional radio bridges (PtP/PtMP)** — for stationary **Edge AI nodes and remote zones where line-of-sight exists**: a live (real-time) high-throughput channel without a monthly cellular bill, **available already in the MVP** (fixed infrastructure from day 1, does not wait for the shuttle).

**Channel selection criterion = data class × availability of line-of-sight × heritage constraints:** LoS exists and real-time is needed at a stationary point → **radio bridge** (cheaper than cellular to operate); no LoS / cannot mount infrastructure on protected buildings / heavy delay-tolerant (360 video) → **data mule**; real-time where there is neither LoS nor a bridge → **cellular**. The more sprawling the estate, the more radio bridges (stationary) and data mules (delay-tolerant) win against linear cellular.

## Consequences
- Pros: optimum of cost/coverage/reliability; no capex for a gateway at every enclosure; a radio bridge gives real-time to remote enclosures **in the MVP** without a monthly cellular bill; critical alerts go without delay; scales with area.
- Cons: several channels complicate operations and monitoring; the data mule introduces latency (minutes) and depends on shuttle routes (R3). **Radio bridges require line-of-sight**, which is broken in a park/zoo by: buildings, rides, metal structures, trees, enclosures, and artificial rocks; **growth and seasonality of vegetation** (in winter the channel is OK, a year later foliage jams it); **weather** (downpour/fog/humidity, stronger on long links); **new objects** on continuously developing territory (a pavilion/stage/scenery stands in the beam); **failure of one bridge** disconnects an entire zone if it is the only one; **alignment** drifts after wind/repair/mast replacement.
- Mitigations: a unified "data transport" abstraction over the channels; at-least-once + idempotency (see [ADR-007](ADR-007-event-reliability-inbox-outbox.md)); for enclosures off the route — +1 utility mule cart (assumption). **For radio bridges:** a **site survey** before deployment (LoS + Fresnel + margin for vegetation growth and construction); **redundancy/mesh** for critical links (not the only path to a zone); **local buffering** on the edge (rides out degradation); the option to **replace the link with fiber** for critical zones; periodic re-alignment in the operations regulation.

## Considered / Rejected alternatives
- **Only fixed Wi-Fi/mesh across the whole territory** — rejected: prohibitively expensive to run over "large and sprawling" land; dead zones would remain anyway.
- **Cellular uplink on each of the 55 enclosures** — rejected: linear capex/opex (≈$1,320/enclosure over 3 years); at N ≥ ≈2 the data mule is cheaper; justified only pointwise for real-time-critical enclosures without LoS.
- **Only radio bridges across the whole territory** — rejected as the sole channel: LoS cannot be built to all 55 enclosures (foliage, rocks, buildings), and a single radio channel = SPOF; a radio bridge is one of the hybrid's channels, not a replacement for it.

## Traceability
Brief goal "delivery of data from the estate to the cloud under patchy Wi-Fi" + "large and sprawling" → offline-tolerance·Reliability·Cost → hybrid connectivity with an area threshold. Related ADRs: [ADR-003](ADR-003-edge-mqtt-store-and-forward.md), [ADR-007](ADR-007-event-reliability-inbox-outbox.md). Threshold rationale — `../03-views-and-perspectives/deployment.md`.

**Mitigates risks:** R3, R21 (see [risk-register](../03-views-and-perspectives/risk-register.md)).
