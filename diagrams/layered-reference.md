# Layered logical view (reference spine)

**What it shows:** a logical representation of the solution as a layered stack (IoT → bus → services → lakehouse → models → agents) on top of an event-driven edge architecture.

A logical representation of the solution as a layered stack. **This is a logical view** — physically the architecture is event-driven + edge (data also travels bypass routes: edge buffering, data mule, real-time bypass — see `deployment.md`, `reliability.md`), so strict linearity here is a convention.

```mermaid
flowchart TB
    L1["1 · IoT Sensors (MQTT)<br/>Q4 enclosure sensors · Fixed Cameras (edge-CV) · Q2 gate · Q10 shuttle"]
    L2["2 · Event Platform<br/>bus: topics/compacted · Inbox/Outbox · partition key"]
    L3["3 · Operational Services (by business capability)<br/>Ticketing/Membership (Q1) · Zoo/Animal Care (Q4/Q5) · Ride Mgmt/Attractions (Q13) · Growth (Q6) · Notif (Q7) · Identity (Q8) · Feedback (Q9) · Marketing (Q11) · Ops (Q12)"]
    L4["4 · Lakehouse + Feature Store + Semantic Layer<br/>medallion (bronze/silver/gold) · CDC from the bus · features/lineage · business metrics (ADR-022)"]
    L5["5 · AI models (behind the AIP)<br/>Crowd · Animal Health · Pricing · CV · guardrails · eval · registry (ADR-008)"]
    L6["6 · AI Agent Platform → agents<br/>Visitor · Operations · Animal · Management · Autonomous Shuttle (ADR-023/024)"]

    L1 --> L2 --> L3 --> L4 --> L5 --> L6
    L3 -. "edge inference and real-time bypass<br/>(patchy Wi-Fi)" .-> L5
    L6 == "tools (least-privilege) → actions" ==> L3
    L6 == "↘ read/write shared memory" ==> L4
    L5 -. "online features" .-> L4
```

## Agents by stakeholder + shared memory (enhanced view)

```mermaid
flowchart TB
    va["Visitor Agent<br/>(visitors)"]
    oa["Operations Agent<br/>(staff)"]
    aa["Animal Agent<br/>(veterinarians)"]
    ma["Management Agent<br/>(management)"]
    lake[("Lakehouse + Feature Store + Semantic Layer<br/>= Digital Twin: domain projections/aggregates<br/>long-term memory · features · lineage")]
    work[("Fast memory<br/>vector KB + session/state")]
    ops["Operational quanta<br/>Q1/Q3/Q5/Q6/Q7/Q12"]

    va -->|tools| ops
    oa -->|tools| ops
    aa -->|tools| ops
    ma -->|tools| ops
    va -.context.-> work
    oa -.context.-> work
    aa -.context.-> work
    ma -.context.-> lake
    work -.populated.-> lake
    va -.outcome memory.-> lake
    oa -.outcome memory.-> lake
    aa -.outcome memory.-> lake
    ops -.CDC/events.-> lake
```

**The idea:** AI is embedded in business processes **by role** (visitors, staff, veterinarians, management), and the Lakehouse is a **shared memory and analytical layer**, not a separate warehouse with ML on the side. Working memory is a fast tier (ADR-024), operations go through tools into the quanta (ADR-023).

**Digital Twin** is a **projection**, not a separate component: a set of up-to-date domain projections/aggregates (gold + Semantic Layer) on top of the Lakehouse, giving agents a holistic picture of the park's state for decisions. We do not introduce a separate Twin layer/quantum.

## Legend / mapping
| Layer | Our components |
|---|---|
| 1 IoT Sensors (MQTT) | Q4, Fixed Cameras, Q2, Q10 (edge, store-and-forward, data mule) |
| 2 Event Platform | the event bus (Inbox/Outbox, compacted topics, partition key) |
| 3 Operational Services | Q1 Ticketing/Membership · Q4/Q5 Zoo & Animal Care · **Q13 Ride & Attractions Mgmt** · Q6, Q7, Q8, Q9, Q11, Q12 |
| 4 Lakehouse & Feature Store & **Semantic Layer** | ADR-022 (CDC on top of database-per-quantum) + business metrics for NL analytics |
| 5 AI models (behind the AIP) | Crowd/Animal/Pricing/CV — inference models (ADR-008) |
| 6 AI Agent Platform → agents | ADR-023/024 (Visitor/Operations/Animal/Management) + the physical shuttle (Q10) |

**Client channels:** Visitor Web/Mobile App (PWA) — visitors; **Staff Portal** — staff/veterinarians/management (host of the Operations/Animal/Management agents).

**Difference from a pure pipeline (our strength):** AI also runs on the edge (L5 ↔ L3), critical data bypasses the layers, and agents act through typed tools back into the operational services. This gives resilience to patchy Wi-Fi, which a linear stack does not provide.
