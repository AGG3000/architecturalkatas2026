# Diagrams

Each diagram is simple (boxes/arrows/words), with a **legend** and a **text description** in its own `.md`. Sources are Mermaid; ready **SVG exports** are in [`exports/`](./exports/) (name = source name: `c4-container.md` → `exports/c4-container.svg`; `layered-reference` has two blocks → `-1`/`-2`).

## Diagram set
- [x] **C4 L1 Context** — the estate and external actors/systems. → [`c4-context.md`](./c4-context.md)
- [x] **C4 L2 Containers** — quanta + event backbone + edge layer + AIP + **data platform (Lakehouse / Feature Store / Semantic Layer) and agent layer**. → [`c4-container.md`](./c4-container.md)
- [x] **Sequence: ticket purchase** — visitor→BFF→Q1→PSP→confirmation. → [`sequence-ticket-purchase.md`](./sequence-ticket-purchase.md)
- [x] **Sequence: animal health-alert** — edge-CV→backbone→Q5→human-in-the-loop→Q7. → [`sequence-health-alert.md`](./sequence-health-alert.md)
- [x] **Sequence: route-quest step** — app→BFF→Q6, queues from Q3, reward via Q1. → [`sequence-route-quest.md`](./sequence-route-quest.md)
- [x] **Deployment** — edge/cloud/network + channels (Wi-Fi/mesh/data mule/cellular). → [`deployment.md`](./deployment.md)
- [x] **AI Gateway** — Prompt Adapter · Model Router · Eval · Registry → providers. → [`ai-gateway.md`](./ai-gateway.md)
- [x] **Layered view (reference spine)** — IoT→Event→Lakehouse→AI→Agents→Decisions + agents by stakeholder ("digital twin" = gold+Semantic projections, not a separate layer). → [`layered-reference.md`](./layered-reference.md)

### Data-flow (data movement and use)
- [x] **Sensor → decision loop** (flagship) — edge→backbone→Q5→lakehouse (domain projections)→agent→human. → [`flow-sensor-to-decision.md`](./flow-sensor-to-decision.md)
- [x] **Data-mule** — enclosure buffer→shuttle→dump at charging→lakehouse (patchy Wi-Fi). → [`flow-data-mule.md`](./flow-data-mule.md)
- [x] **Data→forecast→action** — Crowd Prediction: cameras→Q3→feature store→Q6/Q12/Q7. → [`flow-data-to-decision.md`](./flow-data-to-decision.md)
- [x] **Data lifecycle (medallion)** — event→CDC→bronze/silver/gold→feature/semantic→projections (twin)→models/NL. → [`flow-data-lifecycle.md`](./flow-data-lifecycle.md)

> Each file contains H1 + diagram (Mermaid) + legend + text description of elements/flows; the ready SVG — in [`exports/`](./exports/).

The diagrams are self-contained: the entire source (the main quanta diagram, the mobile layer, cameras, Q10, Q11) is described by mermaid blocks directly in the files of this folder — no external dependencies.

> **C4 L3 (component) — deliberately omitted.** The quanta are team-sized, their internal structure is typical (API/handlers/domain/adapters), and a component decomposition of each quantum would bloat the set without new substance. Instead of L3 we provide: **dynamics** (sequence + data-flow of key scenarios) and a **component cross-section of the core container** — the internals of the AI Gateway ([`ai-gateway.md`](./ai-gateway.md)). If needed, L3 can be derived by the same pattern for a specific quantum.
