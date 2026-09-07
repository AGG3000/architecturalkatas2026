# ADR-022: Lakehouse and Feature Store on top of database-per-quantum

## Status
Accepted

## Context
The AI domains (Crowd Prediction [ADR-020](ADR-020-crowd-prediction-ai.md), Dynamic Pricing [ADR-021](ADR-021-dynamic-pricing-ai.md), Animal Health Q5, return forecast Q6) need **historical data, unified features, training data, and lineage**. Currently data sits under the **database-per-quantum** principle ([ADR-002](ADR-002-quantum-boundaries-by-characteristics.md)) + a separate warehouse (Q3) and time-series store (Q4) — **fragmented**: there is no single place for ML data/features, there is duplication, and there is no lineage or consent-for-training. Abandoning database-per-quantum is not an option — that is the autonomy of the quanta. We need to **combine** operational autonomy with a unified analytics-ML platform.

## Decision
Introduce a **Lakehouse** (medallion: bronze→silver→gold, batch + streaming) and a **Feature Store**, populated from the quanta's operational DBs via a **CDC/event stream on top of the existing bus** — without touching operational autonomy. Separation of responsibility: the quanta's operational DBs are the source of truth for **operations**; the lakehouse is the source of truth for **analytics/ML**. The Feature Store provides **unified online/offline features** for Crowd Prediction, Pricing, Health, and the return forecast. Lineage and data governance (including consent-for-training) live at the lakehouse level (tied to [ADR-018](ADR-018-ai-governance-framework.md)).

## Consequences
- Pros: unified training data and features, less duplication, consistency of online/offline features, lineage/governance in one place; feeds all AI domains; removes the contention of ML queries with the operational DBs.
- Cons: an extra platform (cost/operation); ETL/CDC introduces delay (eventual); a "data swamp" risk without discipline.
- Mitigations: a medallion structure + owners of domain datasets; CDC from the event bus (reusing the backbone, [ADR-007](ADR-007-event-reliability-inbox-outbox.md)); governance hooks (consent/lineage); analytics is still the read-side ([ADR-005](ADR-005-cqrs-analytics-operations-split.md)).

## Considered / Rejected alternatives
- **Only database-per-quantum + direct ML queries to operational DBs** — rejected: resource contention, no unified features/lineage, duplicates (the same reason as in [ADR-005](ADR-005-cqrs-analytics-operations-split.md)).
- **A single DB for everything instead of database-per-quantum** — rejected: it wrecks quanta autonomy ([ADR-002](ADR-002-quantum-boundaries-by-characteristics.md)), strong coupling, a shared failure.

## Traceability
AI domains (§D/§G) → Analyzability/Evolvability + data governance → a Lakehouse + Feature Store via CDC **on top of** database-per-quantum. Details — `../03-views-and-perspectives/data.md`. Related ADRs: [ADR-002](ADR-002-quantum-boundaries-by-characteristics.md), [ADR-005](ADR-005-cqrs-analytics-operations-split.md), [ADR-007](ADR-007-event-reliability-inbox-outbox.md), [ADR-018](ADR-018-ai-governance-framework.md), [ADR-020](ADR-020-crowd-prediction-ai.md), [ADR-021](ADR-021-dynamic-pricing-ai.md).

**Mitigates risks:** R4 (see [risk-register](../03-views-and-perspectives/risk-register.md)).
