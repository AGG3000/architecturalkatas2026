# ADR-015: Autonomous Shuttle — supervised → driverless (buy-and-integrate) + data mule (DTN)

## Status
Accepted *(decision accepted; rollout — roadmap R3)*

## Context
A "large and sprawling" territory (fact), 40 rides + 55 enclosures, and growth from 5000→15000 visits/day make internal transport a real need. Drivers are an operating expense that undermines profitability (the brief's goal). **Patchy Wi-Fi** forbids cloud real-time control and requires "a way to deliver data to the cloud" (facts). The context is safety-critical and hazardous: crowds, **children**, and nearby **venomous/exotic animals**. Affected: **Safety, offline-tolerance, Reliability, Cost**.

## Decision
A dual role for quantum Q10. (1) **Autonomy:** **buy** a certified low-speed platform (Navya/EasyMile class — assumption) and integrate it behind a provider abstraction; at MVP — **supervised** (geo-fenced low-speed routes, safety driver + remote teleoperation, **fail-safe to stop** on uncertainty); driverless is a roadmap phase after V&V statistics. Autonomy is computed **on board** (edge). (2) **Data mule (DTN):** as it passes, the shuttle picks up the sensor buffer of remote enclosures (MQTT/BLE/LoRa) and idempotently uploads it to the cloud at a stop with connectivity. Data is strictly split: **real-time-critical** → fixed/cellular uplink; **delay-tolerant** → data mule.

## Consequences
- + A genuine AI application; reduced staffing costs, 24/7 operation; cheaply covers remote enclosures without capex on 55 fixed gateways; synergy (the shuttle self-positions).
- − The hardest AI to verify; liability/insurance/regulation (amplified by children and venomous animals); **provider lock-in is harsher than usual** (changing vendor = changing the physical platform); capex/leasing on hardware; the data-mule delivery delay is unacceptable for urgent data.
- Mitigation: ODD + teleoperation + shadow-mode + perception fitness functions; cellular fallback for the critical class; at-least-once + idempotency by event id; buffer encryption.

## Considered / Rejected alternatives
- **Full driverless from day one** — rejected: an unacceptable risk/liability profile in a context with children and venomous animals, weak V&V at launch.
- **Build the autopilot from scratch** — rejected: unrealistic on timeline/budget, the wrong level of detail.
- **Cellular uplink at every enclosure** — rejected: reliable, but expensive in capex/opex for 55 points; justified only for real-time-critical enclosures.

## Traceability
Brief goal (profitability, data delivery under patchy Wi-Fi) → **Safety/offline-tolerance/Cost** → buy supervised autonomy + data mule with data classification (Provider pattern, at-least-once + idempotency by event id). Related ADRs: [ADR-011](ADR-011-fixed-cameras-edge-cv-vs-cloud-cv.md), [ADR-014](ADR-014-mobile-client-pwa-vs-native.md), AIP.

**Mitigates risks:** R11, R19 (see [risk-register](../03-views-and-perspectives/risk-register.md)).
