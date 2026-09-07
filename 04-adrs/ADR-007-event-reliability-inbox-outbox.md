# ADR-007: Event reliability — Inbox/Outbox + at-least-once + a single partitioning key

## Status
Accepted

## Context
The event backbone (see [ADR-001](ADR-001-event-driven-architecture-and-quanta.md)) and store-and-forward under patchy Wi-Fi (see [ADR-003](ADR-003-edge-mqtt-store-and-forward.md), [ADR-004](ADR-004-edge-connectivity-mesh-cellular-data-mule.md)) give rise to classic problems: **dual write** (you cannot atomically save and publish an event), **duplicates** under at-least-once delivery and repeated unloading of data-mule buffers, **races** of updates to a single record. For tickets/payments a double charge is unacceptable (see [ADR-006](ADR-006-pci-ticketing-isolation-external-psp.md)); for telemetry, loss of events is unacceptable (brief fact: monitoring of animal health, §D3). These are implementation assumptions on top of brief facts.

## Decision
Standardize event reliability across the whole system: (1) **Outbox** on the producer side and **Inbox** on the consumer side (the Inbox/Outbox pattern) against dual-write; (2) **at-least-once delivery + idempotency** by unique event id + version against duplicates and stale events; (3) a **single partitioning key** for a paired topic and table, so that all updates of one record go sequentially to one partition. Rationale: without these patterns, event decoupling produces losses, duplicates, and races.

## Consequences
- Pros: guarantee of delivery at least once, safe retries, idempotency (repeated unloading of the data mule is harmless), ordering of updates by key, protection against double payments.
- Cons: the Inbox requires polling, additional processes and tables complicate the system, the partitioning key may become suboptimal over time (risk of hotspots/fragmentation).
- Mitigations: monitoring of Inbox/Outbox lag; event versioning to discard stale ones; revising the key as load grows; compacted topics for a state snapshot (assumption).

## Considered / Rejected alternatives
- **Direct publication to the broker from the business transaction (no Outbox)** — rejected: dual-write loses the event on a failure between the DB commit and publication; unacceptable for payments and telemetry.
- **Exactly-once at the broker level** — rejected: expensive, fragile, and poorly survives store-and-forward/repeated unloading; at-least-once + idempotency is simpler and more reliable in the edge context.

## Traceability
Brief goal "reliable animal monitoring + tickets under patchy Wi-Fi" → characteristics Reliability·Consistency(in Q1) → Inbox/Outbox + at-least-once + a single partitioning key. Related ADRs: [ADR-001](ADR-001-event-driven-architecture-and-quanta.md), [ADR-003](ADR-003-edge-mqtt-store-and-forward.md), [ADR-004](ADR-004-edge-connectivity-mesh-cellular-data-mule.md), [ADR-005](ADR-005-cqrs-analytics-operations-split.md), [ADR-006](ADR-006-pci-ticketing-isolation-external-psp.md).

**Mitigates risks:** R2 (see [risk-register](../03-views-and-perspectives/risk-register.md)).
