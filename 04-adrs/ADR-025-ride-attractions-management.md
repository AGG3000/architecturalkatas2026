# ADR-025: Ride & Attractions Management domain (Q13)

## Status
Accepted

## Context
The territory holds **40 18th-century rides** (brief fact, §C) that have "passed safety inspections" (§B). Until now the rides have appeared only as a fact and as "maintenance windows" inside Q12, but there was **no separate ride-management domain**. Yet it is a standalone business capability with its own characteristics: status/availability, throughput, **periodic safety inspections** (historical structures → critical), predictive maintenance by cycles. Mixing this with scheduling (Q12) or analytics (Q3) blurs the boundaries.

## Decision
Carve out a **quantum Q13 — Ride & Attractions Management**: a ride registry, status/availability (open/closed/maintenance), capacity and throughput, a **safety-inspection** log with reminders, and **predictive maintenance** on **MQTT mechanism telemetry** (using the §E budget). **We collect from each ride:** vibration/accelerometers · temperature (bearings/motors) · energy consumption/current · cycle counter · speed · controller errors · state of the braking systems. **Three AI scenarios:** (1) **anomaly detection** — a model knows the mechanism's normal behavior and catches deviations by vibration/thermal/current signatures; (2) **remaining useful life (RUL) estimation**; (3) **failure forecasting** before it occurs → an early maintenance window. *Example:* a joint **rise in vibration + rise in temperature + increase in energy consumption** of a node → the model outputs "**high probability of a bearing failure within the next ~14 days**" → the ride is scheduled for maintenance **by condition** (not by calendar), before failure. Additionally — **thermal-imaging control of mechanism overheating** (bearings/motors/brakes): abnormal heating → an early maintenance signal (thermal is apt here — metal gives a thermal contrast). **Safety invariant:** AI **flags** a ride for inspection/restriction, but **only a human clears it for operation** — AI does not "certify" soundness. Integrations via the bus: statuses/closures → Q3 (analytics) and Q6 (the route-quest bypasses closed ones), incidents/overdue inspections/overheating → Q7 (alert to staff), maintenance windows are coordinated with Q12, CV inspection of structures and thermal overheating — with fixed cameras. Characteristics: Safety · Availability · Reliability.

## Consequences
- Pros: explicit ownership of ride status/safety; the route-quest does not lead into closed ones; an inspection audit (compliance with requirements). **The predictive-maintenance business case:** fewer breakdowns + fewer unplanned downtimes + shorter queues (a closed ride overloads the neighbors) → higher availability and **revenue** — a direct link Q13 → Q3 (queues) → Q6 (route) and the overall revenue thesis.
- Cons: yet another quantum (complexity); predictive maintenance requires cycle history (cold-start).
- Mitigations: at MVP, status/inspections are a simple registry + rules; predictive maintenance under the "rules→AI" strategy ([ADR-009](ADR-009-rules-to-ai-cold-start.md)); in the MVP rollup it may start as a module next to Q12 and split off as it grows. **Sensor validation:** the "normal-behavior" baseline is built on the mechanism's **run-in** data; **self-test + redundancy + a sensor health-check** separate *sensor* drift from *mechanism* drift (otherwise a corrupt vibration signature wrecks recall-0.95); calibration per regulation.

## Considered / Rejected alternatives
- **Keep rides inside Q12 (Scheduling)** — rejected: rides have their own characteristics (Safety/Availability) and a status lifecycle, not just maintenance windows; mixing blurs the boundaries.
- **Reduce it to queue analytics (Q3)** — rejected: Q3 measures demand but does not manage ride status/safety/maintenance.

## Traceability
§C (40 rides) + §B (safety inspections) + §D (popularity/growth) → Safety/Availability/Reliability → a separate ride-management domain. Related ADRs: [ADR-009](ADR-009-rules-to-ai-cold-start.md), [ADR-016](ADR-016-ai-marketing-and-scheduling-hitl-provider-abstraction.md) (Q12 maintenance windows), [ADR-005](ADR-005-cqrs-analytics-operations-split.md) (Q3), [ADR-020](ADR-020-crowd-prediction-ai.md) (queues), [ADR-011](ADR-011-fixed-cameras-edge-cv-vs-cloud-cv.md) (CV inspection).

**Mitigates risks:** R12 (see [risk-register](../03-views-and-perspectives/risk-register.md)).
