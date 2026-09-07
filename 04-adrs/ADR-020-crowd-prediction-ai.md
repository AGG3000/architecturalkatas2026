# ADR-020: Crowd Prediction AI — forecasting queues and density ahead

## Status
Accepted

## Context
Brief §D/§F: understand the popularity of zones, deploy staff sensibly, and grow attendance. Q3 **measures current** queues/density (fixed cameras, see [ADR-005](ADR-005-cqrs-analytics-operations-split.md)), but that is not enough for proactivity: for the route-quest (Q6) to route around **future** peaks, for schedules (Q12) to place staff **in advance**, and to warn visitors (Q7) — we need a **forecast of queues/crowds ahead**, not just the fact. At launch there is no history (cold-start, see [ADR-009](ADR-009-rules-to-ai-cold-start.md)). Characteristics: Analyzability, Accuracy, Elasticity.

## Decision
Introduce **Crowd Prediction as an AI capability behind the AIP** on top of Q3 data (not a new heavy quantum): a **time-series/ML forecast** of queue lengths and zone density over a minutes-to-hours horizon from Q3 historical patterns + calendar/events (Q7) + weather/season (an external signal — assumption). Start under the "rules→AI" strategy (heuristics by time of day/day of week) until history accumulates. Outputs: signals into **Q6** (proactive route), **Q12** (rostering), **Q7** (push "a queue will grow soon / it's free now").

## Consequences
- Pros: proactivity (bypassing peaks, staff in advance, shorter queues → better UX and return); reuses already-collected Q3 data.
- Cons: accuracy depends on history/seasonality; a **risk of a self-fulfilling forecast** (everyone is sent to an "empty" zone → it fills up).
- Mitigations: a closed loop with frequent updating and forecast smoothing; soft distribution (don't drive everyone into one zone); a fitness function — queue forecast error vs actual (threshold, drift).

## Considered / Rejected alternatives
- **Measuring current queues only (Q3, reactive)** — rejected: we react late, the peak has already happened; no proactive routing/deployment.
- **Static rules without ML** — rejected as the sole solution: they don't catch seasonality/the event effect; suitable only as a cold-start baseline.

## Traceability
§D (zone popularity) + §F (where to invest/staff, growth) → Analyzability/Accuracy/Elasticity → forecasting queues ahead, feeding the route and schedules. Related ADRs: [ADR-005](ADR-005-cqrs-analytics-operations-split.md), [ADR-009](ADR-009-rules-to-ai-cold-start.md), [ADR-016](ADR-016-ai-marketing-and-scheduling-hitl-provider-abstraction.md) (Q12), [ADR-021](ADR-021-dynamic-pricing-ai.md) (demand → price).

**Mitigates risks:** R10 (see [risk-register](../03-views-and-perspectives/risk-register.md)).

**V&V:** forecast accuracy (queue error vs actual, drift) — specific metrics/thresholds and the response in [validation & verification](../05-ai/validation-verification.md).
