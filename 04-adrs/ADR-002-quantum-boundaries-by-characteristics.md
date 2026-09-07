# ADR-002: Quantum boundaries drawn along divergent quality characteristics

## Status
Accepted

## Context
After choosing the event-driven style (see [ADR-001](ADR-001-event-driven-architecture-and-quanta.md)), we must decide **where to cut the boundaries**. The estate's domains require conflicting properties: payments — Security/PCI and consistency (brief fact: tickets and family passes, §D1); entry scanners and IoT ingestion — operation offline under patchy Wi-Fi (brief fact, §E); zone popularity analytics — read scale and analyzability (brief fact, §D2); animal monitoring — throughput and accuracy (brief fact, §D3). Merging them into one quantum means imposing the strictest requirement of each on everyone and creating competition for resources (assumption).

## Decision
Draw quantum boundaries **where the driving characteristics diverge** (boundaries by divergent characteristics). Hence, in particular, the deliberate separation of Q1 Ticketing (PCI/consistency), Q2 Gate and Q4 IoT (offline-tolerance), Q3 Analytics (read-side scale) as distinct quanta with their own databases. Each quantum is a unit of independent deployment with its own top-3 characteristics; between quanta — only events.

## Consequences
- Pros: isolation of failures and requirements (PCI does not "leak" into analytics), independent scaling to the load profile, clean evolution of each domain.
- Cons: more moving parts and databases, duplication of some data, requires discipline of event contracts and end-to-end observability.
- Mitigations: MVP consolidation (6-7 quanta + AIP) with an explicit roadmap for splitting on a load/team/evolution signal; the non-shareable core (Ticketing, edge/offline, Animal, Analytics) is kept separate from day one (assumption).

## Considered / Rejected alternatives
- **Boundaries by technical layers (UI/logic/data)** — rejected: layers do not isolate divergent characteristics; a change in one domain touches all layers, PCI and offline are smeared across the whole system.
- **One large "Park Platform" quantum** — rejected: imposes the maximum of requirements (PCI + offline + read-scale) on all domains, makes independent scaling impossible, and turns the bus into an internal call.

## Traceability
Brief goal "understand zone popularity / tickets / animal monitoring under patchy Wi-Fi" → divergent characteristics Security·Consistency vs offline-tolerance vs Scalability → quantum boundaries by characteristics. Related ADRs: [ADR-001](ADR-001-event-driven-architecture-and-quanta.md), [ADR-003](ADR-003-edge-mqtt-store-and-forward.md), [ADR-005](ADR-005-cqrs-analytics-operations-split.md), [ADR-006](ADR-006-pci-ticketing-isolation-external-psp.md).
