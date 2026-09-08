# ADR-015: Autonomous Shuttle — supervised → driverless (buy-and-integrate) + data mule (DTN)

## Status
Accepted *(asset — MVP, manned electric; autonomy rollout — roadmap R3)*

## Context
A "large and sprawling" territory (fact), 40 rides + 55 enclosures, and growth from 5000→15000 visits/day make internal transport a real need. Drivers are an operating expense that undermines profitability (the brief's goal). **Patchy Wi-Fi** forbids cloud real-time control and requires "a way to deliver data to the cloud" (facts). The context is safety-critical and hazardous: crowds, **children**, and nearby **venomous/exotic animals**. Affected: **Safety, offline-tolerance, Reliability, Cost**.

## Decision
We split the **asset** from the **autonomy** for quantum Q10. **MVP:** a **manned electric land-train** (off-the-shelf, leased — low risk) that provides passenger transport (premium/access/accessibility) AND carries the **data-mule collector from day 1**. **Roadmap R3 (autonomy):** acquire/integrate a certified low-speed autonomous platform (Navya/EasyMile class — assumption) behind a provider abstraction; **supervised** first (geo-fenced low-speed routes, safety driver + remote teleoperation, **fail-safe to stop** on uncertainty), then **driverless** after V&V statistics. Autonomy is computed **on board** (edge). **Data mule (DTN):** works **from the MVP on the manned vehicle** — as it passes it picks up the sensor buffer of remote enclosures (MQTT/BLE/LoRa) and idempotently uploads it to the cloud at a stop with connectivity. Data is strictly split: **real-time-critical** → fixed/cellular uplink; **delay-tolerant** → data mule.

## Consequences
- + **Transport + data-mule available from day 1** (manned, off-the-shelf) — no waiting for autonomy V&V; **accessibility** for elderly/disabled/families and access to remote zones.
- + A genuine AI application (at R3); reduced staffing costs, 24/7 operation; cheaply covers remote enclosures without capex on 55 fixed gateways; synergy (the shuttle self-positions).
- − Autonomy (R3) is the hardest AI to verify; liability/insurance/regulation (amplified by children and venomous animals); **provider lock-in is harsher than usual** (changing vendor = changing the physical platform); capex/leasing on hardware; the data-mule delivery delay is unacceptable for urgent data.
- − The manned transport carries **driver OpEx** until autonomy (R3) removes it.
- Mitigation: ODD + teleoperation + shadow-mode + perception fitness functions; cellular fallback for the critical class; at-least-once + idempotency by event id; buffer encryption.

## Considered / Rejected alternatives
- **Full driverless from day one** — rejected: an unacceptable risk/liability profile in a context with children and venomous animals, weak V&V at launch.
- **Defer all transport to the roadmap** — rejected: the sprawling estate needs transport (access/accessibility/remote data pickup) from day 1; only the *autonomy* needs deferral for V&V/regulatory reasons.
- **Build the autopilot from scratch** — rejected: unrealistic on timeline/budget, the wrong level of detail.
- **Cellular uplink at every enclosure** — rejected: reliable, but expensive in capex/opex for 55 points; justified only for real-time-critical enclosures.

## Traceability
Brief goal (profitability, data delivery under patchy Wi-Fi) → **Safety/offline-tolerance/Cost** → buy supervised autonomy + data mule with data classification (Provider pattern, at-least-once + idempotency by event id). Related ADRs: [ADR-011](ADR-011-fixed-cameras-edge-cv-vs-cloud-cv.md), [ADR-014](ADR-014-mobile-client-pwa-vs-native.md), AIP.

**Mitigates risks:** R11, R19 (see [risk-register](../03-views-and-perspectives/risk-register.md)).
