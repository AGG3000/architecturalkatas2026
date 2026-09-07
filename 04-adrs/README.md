# Architecture Decision Records (index)

26 active ADRs (+ ADR-012 merged into ADR-011) in the Nygard format (Status · Context · Decision · Consequences · Considered/Rejected · Traceability). Template — [`000-template.md`](000-template.md). ⭐ — key decisions (see also "Key Architectural Decisions" in the root README).

## Style and boundaries
- ⭐ [ADR-001](ADR-001-event-driven-architecture-and-quanta.md) — Event-driven backbone + architectural quanta
- [ADR-002](ADR-002-quantum-boundaries-by-characteristics.md) — Quantum boundaries by diverging characteristics

## Edge and connectivity (patchy Wi-Fi)
- [ADR-003](ADR-003-edge-mqtt-store-and-forward.md) — Edge + MQTT store-and-forward
- ⭐ [ADR-004](ADR-004-edge-connectivity-mesh-cellular-data-mule.md) — Edge connectivity: hybrid mesh / cellular / data-mule (threshold by area)
- [ADR-017](ADR-017-connectivity-failure-degraded-operation.md) — Connectivity failure and degraded mode

## Data and reliability
- [ADR-005](ADR-005-cqrs-analytics-operations-split.md) — CQRS split of analytics and operations
- [ADR-007](ADR-007-event-reliability-inbox-outbox.md) — Event reliability: Inbox/Outbox + partition key
- ⭐ [ADR-022](ADR-022-lakehouse-and-feature-store.md) — Lakehouse + Feature Store on top of database-per-quantum
- [ADR-024](ADR-024-lakehouse-as-agent-memory.md) — Lakehouse as shared agent memory + a working-memory tier

## Security and privacy
- [ADR-006](ADR-006-pci-ticketing-isolation-external-psp.md) — PCI isolation of Ticketing + external PSP
- [ADR-013](ADR-013-privacy-surveillance-and-mobile-telemetry.md) — Privacy: video surveillance + mobile telemetry (crypto-shredding)

## AI platform and governance
- ⭐ [ADR-008](ADR-008-ai-platform-provider-abstraction-fallback-cost.md) — AI Gateway (AIP): provider abstraction, model router, fallback, cost
- [ADR-009](ADR-009-rules-to-ai-cold-start.md) — "Rules → AI" for domains without data at launch
- ⭐ [ADR-010](ADR-010-human-in-the-loop-confidence-thresholds.md) — Human-in-the-loop + confidence thresholds
- ⭐ [ADR-018](ADR-018-ai-governance-framework.md) — AI Governance framework (risk classification + controls)
- ⭐ [ADR-023](ADR-023-agentic-layer.md) — Agentic layer (tool-using agents by stakeholder)

## AI domains and capabilities
- [ADR-011](ADR-011-fixed-cameras-edge-cv-vs-cloud-cv.md) — Fixed cameras: edge-CV + RGB/Thermal placement *(merges the former ADR-012)*
- [ADR-012](ADR-012-fixed-cameras-rgb-vs-thermal.md) — *(merged into ADR-011)*
- [ADR-016](ADR-016-ai-marketing-and-scheduling-hitl-provider-abstraction.md) — AI Social Marketing (Q11) + AI Scheduling (Q12)
- ⭐ [ADR-019](ADR-019-rag-knowledge-assistant.md) — RAG assistant (knowledge base / staff / support)
- ⭐ [ADR-020](ADR-020-crowd-prediction-ai.md) — Crowd Prediction AI (queue forecasting)
- ⭐ [ADR-021](ADR-021-dynamic-pricing-ai.md) — Dynamic Pricing AI (tickets/passes)

## Client, transport, domains
- [ADR-014](ADR-014-mobile-client-pwa-vs-native.md) — Mobile client: PWA vs Native
- [ADR-015](ADR-015-autonomous-shuttle-and-data-mule.md) — Autonomous shuttle (supervised → driverless) + data mule
- [ADR-025](ADR-025-ride-attractions-management.md) — Ride & Attractions Management (Q13)
- [ADR-026](ADR-026-transport-sizing-and-night-autopilot.md) — Transport: passenger model, data-mule throughput, night autopilot trips for metrics
- [ADR-027](ADR-027-visitor-rfid-token.md) — Visitor RFID token (wristband/pass) in addition to the app
