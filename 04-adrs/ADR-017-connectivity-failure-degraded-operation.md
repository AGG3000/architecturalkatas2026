# ADR-017: Connectivity Failure and Degraded Operation Mode

## Status
Accepted

## Context
The real-time-critical data class (an alert about an escaped/sick venomous animal, fire, a lost-child search, a payment) travels over the fixed/cellular channel (see [ADR-004](ADR-004-edge-connectivity-mesh-cellular-data-mule.md)). The architecture survives **intermittent** connectivity well (store-and-forward, data mule, offline ticket cache — see [ADR-003](ADR-003-edge-mqtt-store-and-forward.md)), but two scenarios are not covered:
1. **Failure of the single mobile carrier** — the cellular channel serves as the fallback for the critical class; one carrier = a single point of failure for the most important data.
2. **Full/prolonged loss of the channel to the cloud** — there is no explicit degraded mode: ticket sales (Q1) require an online PSP; cloud processing of critical alerts (Q5/Q7) is unavailable.

Brief §E: patchy Wi-Fi + a channel to the cloud is required. We need a plan not only for intermittency, but also for **channel/carrier failure**.

## Decision
1. **Redundancy of the critical path:** dual-carrier (2 SIMs/carriers), optionally a satellite backup (Starlink-class — assumption) for the safety-critical uplink. Failure of one carrier does not break the critical class.
2. **Local emergency alerting:** at critical enclosures — **deterministic edge rules** (threshold → local siren / notification to the keeper over LAN/mesh), triggering **independently of the cloud**; the cloud/AI only enrich, but the baseline safety alert is local.
3. **Offline sales mode:** kiosks cache the order and **defer the payment** (store-and-forward of orders, idempotency, limits against fraud); validation of already-purchased tickets always works offline (Q2). The acceptable minimum is fail-closed (prepaid/family-pass only) during a prolonged outage.
4. **Degradation matrix** — `../03-views-and-perspectives/reliability.md`.

## Consequences
- Pros: no single-carrier SPOF for the critical class; safety does not depend on the cloud; the estate keeps operating in degraded mode; delay-tolerant data is not lost (buffer).
- Cons: added cost (a second carrier/satellite); the complexity of the offline payment queue (risk of duplicates/fraud); critical rules maintained in two places (edge + cloud).
- Mitigations: order idempotency + offline sales limits; simple deterministic edge rules (not AI) for safety; periodic reconciliation of edge rules with the cloud ones.

## Considered / Rejected alternatives
- **Relying on a single cellular carrier only as the fallback** — rejected: a single-carrier SPOF for the most important data class.
- **Full offline autonomy of the whole estate** (duplicate all functions at the edge) — rejected: expensive and redundant; most functions tolerate delay and are covered by the buffer.
- **Hard fail-closed on all sales during an outage** — considered: simpler and safer against fraud, but loses revenue at peak; a compromise was chosen — a limited offline mode with caps, fail-closed as the last-resort minimum.

## Traceability
Brief §E (patchy Wi-Fi + a channel to the cloud) + §D (tickets, animal health) → **Availability/Reliability/Safety** → redundancy of the critical channel + local safety alerts + an explicit degraded mode. Related ADRs: [ADR-003](ADR-003-edge-mqtt-store-and-forward.md), [ADR-004](ADR-004-edge-connectivity-mesh-cellular-data-mule.md), [ADR-006](ADR-006-pci-ticketing-isolation-external-psp.md), [ADR-015](ADR-015-autonomous-shuttle-and-data-mule.md).

**Mitigates risks:** R1 (see [risk-register](../03-views-and-perspectives/risk-register.md)).
