# ADR-005: CQRS — separating analytics (read-side) from operations (write-side)

## Status
Accepted

## Context
The Countess needs to understand **how popular different parts of the park are**, in order to know where to improve and how to allocate staff (brief fact, §D2, §F). These are heavy aggregations: dwell-time, heatmap, queue/density estimation, NL queries against analytics (assumption). At the same time the operational quanta (tickets, telemetry ingestion) require predictable latency. If analytical aggregations run against operational databases, they compete for resources and take down transactions (assumption). Read-side characteristics (Scalability, Analyzability, Elasticity) diverge from the write-side.

## Decision
Apply **CQRS**: operational quanta write events to the bus (write-side, their own databases), and **Q3 Visitor Movement & Analytics** builds a separate **read-optimized** model (Analytics DB/warehouse) from events. Analytics is an independent quantum with its own database, consuming `ZoneEntered`, `DwellTimeRecorded`, `TicketPurchased` and metadata from edge-CV cameras. Rationale: read and write loads are decoupled, and analytics can be scaled and reshaped to queries without touching operations.

## Consequences
- Pros: heavy aggregations do not affect transactions; the read model is tuned to queries (heatmap/NL); independent scaling; new analytical consumers connect as subscribers.
- Cons: eventual consistency (analytics lags slightly behind operations); data duplication in the read model; complexity of synchronization and handling of duplicate events.
- Mitigations: a single partitioning key for stream↔table against races (see [ADR-007](ADR-007-event-reliability-inbox-outbox.md)); read lag is acceptable for analytics (not real-time); compacted topics for the "latest state" of zones (assumption).

## Considered / Rejected alternatives
- **Shared database for operations and reporting (read-replica of the same store)** — rejected: analytical scans still compete for resources and locks, the write-side schema is not optimal for analytical queries, and they cannot be scaled separately.
- **Direct queries from the analytics frontend to operational services** — rejected: couples analytics to operational contracts and latency, adds load at transaction peaks, no historical depth.

## Traceability
Brief goal "understand zone popularity and where to invest/place staff" → characteristics Scalability·Analyzability·Elasticity (read-side) → CQRS read/write separation. Related ADRs: [ADR-001](ADR-001-event-driven-architecture-and-quanta.md), [ADR-002](ADR-002-quantum-boundaries-by-characteristics.md), [ADR-007](ADR-007-event-reliability-inbox-outbox.md).
