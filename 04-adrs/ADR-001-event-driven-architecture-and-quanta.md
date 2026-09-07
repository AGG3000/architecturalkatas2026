# ADR-001: Event-driven style as the backbone and architectural quanta

## Status
Accepted

## Context
A "new comprehensive modern architecture" is needed (brief fact, §G) for heterogeneous tasks: tickets, zone popularity analytics, animal monitoring, growth of attendance and profit (brief fact, §D). The domains have **sharply divergent quality characteristics**: PCI security and transactional consistency for payments vs offline-tolerance for entry scanners and IoT ingestion under **patchy Wi-Fi** (brief fact, §E). Wi-Fi is uneven, and data must be delivered to the cloud in batches (brief fact, §E). A single synchronous monolith would couple these forces rigidly: heavy analytical aggregations would take down transactions, and a network outage would take down the entire pipeline (assumption).

## Decision
Perform a **worksheet decomposition** (Event Storming → bounded contexts → grouping by the top-3 quality characteristics → quanta) and choose an **event-driven style with a shared bus** as the primary means of communication between quanta; synchronous REST — only for frontend↔BFF. Each quantum is deployed independently and owns its database. Rationale: events decouple domains with different characteristics (scale, offline, consistency), provide buffering under patchy Wi-Fi, and map naturally onto store-and-forward.

## Consequences
- Pros: loose coupling, independent scaling/deployment, buffering during network outages, easy addition of subscribers (analytics, notifications, content).
- Cons: eventual consistency by default, difficulty debugging distributed flows, requires mature observability and explicit handling of duplicates.
- Mitigations: strong consistency only in Ticketing (see [ADR-006](ADR-006-pci-ticketing-isolation-external-psp.md)); Inbox/Outbox + at-least-once (see [ADR-007](ADR-007-event-reliability-inbox-outbox.md)); we start with modular monoliths along quantum boundaries and split off services on a load signal (assumption).

## Considered / Rejected alternatives
- **Monolith / synchronous client-server style** — rejected: couples domains with divergent characteristics, heavy analytics competes with payments for resources, a network outage breaks the entire request; no natural buffering under patchy Wi-Fi.
- **Orchestrated microservices with synchronous calls (no bus)** — rejected: chains of synchronous calls are fragile under patchy Wi-Fi and reduce availability; we lose the decoupling and event buffering.

## Traceability
Brief goal "comprehensive architecture for heterogeneous tasks" + constraint "patchy Wi-Fi" → characteristics Scalability/Availability/offline-tolerance/Consistency → choice of event-driven backbone and division into quanta by characteristics. Related ADRs: [ADR-002](ADR-002-quantum-boundaries-by-characteristics.md), [ADR-003](ADR-003-edge-mqtt-store-and-forward.md), [ADR-007](ADR-007-event-reliability-inbox-outbox.md).
