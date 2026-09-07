# Domain map (quanta)

A summary of the decomposition into architectural quanta. Boundaries are drawn where the **driving characteristics diverge** (each quantum has its own style and its own DB). **13 is the conceptual granularity; the MVP ships ~7 deployable units** — some quanta co-deploy as modular monoliths (Q5 in Q4, Q9 in Q6) until R2 (see the focus note below). Flow diagrams are in `../03-views-and-perspectives/`, decisions are in `../04-adrs/`.

| # | Quantum | Responsibility | Top-3 characteristics | Style | Data ownership |
|---|---|---|---|---|---|
| Q1 | Ticketing & Payments | tickets, family passes, PCI; **dynamic pricing (AI, within corridors)** | Security · Availability · Consistency (transactions) | service / modular monolith + external PSP | Orders/Payments DB (isolated, PCI) |
| Q2 | Access & Gate | ticket scanning, offline | Availability · Performance · offline-tolerance | edge + event-driven | local cache of valid tickets on the gate device |
| Q3 | Visitor Movement & Analytics | movements, queues/heatmap (fixed cameras); **queue forecast ahead (Crowd Prediction AI)** | Scalability · Analyzability · Elasticity | event-driven + read-side (CQRS) | Analytics DB / warehouse (read-optimized) |
| Q4 | Animal IoT Monitoring | MQTT climate sensors (air t°/humidity/CO₂/NH₃/light/UV; water t°/pH/O₂), weight/activity telemetry, **breach sensors + RFID/PIT** (escape/inventory of cold-blooded animals), edge buffer | offline-tolerance · Throughput · Reliability | edge + MQTT ingestion | time-series store (telemetry) |
| Q5 | Animal Health & AI | health/nutrition, piranhas, CV from cameras | Accuracy · Evolvability · AI-provider-agnostic | microservices behind an AI facade | Health/inference results store |
| Q6 | Growth & Recommendation | route quest, AI quests, return forecast | Evolvability · Accuracy · Cost | microservice + AI facade | derived/feature store |
| Q7 | Notifications | pushes/alerts, event announcements | Reliability (delivery) · low-SLA | event-driven | outbox/queue |
| Q8 | Identity & Profile | visitors/staff/roles; **profile + preference store** (preferences → Q6/agents); family linkage; binding of **app + RFID token** to a single profile | Security · Interoperability | buy (external IdP) | Auth + profile/preferences store |
| Q9 | Feedback & Sentiment | feedback + AI sentiment | Evolvability · low-SLA · Analyzability | event-driven + AI facade | Feedback store |
| Q10 | Internal Transport & Mobility | shuttle: autonomy, data mule, mesh, 360, display board | Safety · offline-tolerance · Reliability | mobile edge node + DTN | on-board buffer + transport state store |
| Q11 | AI Social Marketing | highlight-mining from cameras, post generation | Evolvability · AI-provider-agnostic · low-SLA | microservice + social-API provider abstraction | content/asset store, approval-queue |
| Q12 | Operations & Scheduling | staff/feeding/maintenance schedules | Optimizability · Cost · Evolvability (non-real-time) | batch optimization + AI/OR | schedules store |
| Q13 | Ride & Attractions Management | status/availability, capacity, safety inspections, predictive maintenance (40 rides) | Safety · Availability · Reliability | service + event-driven | Attractions registry + inspection/maintenance log |
| AIP | AI Platform (cross-cutting) | provider abstraction, guardrails, eval, cost | Portability · Observability · Cost-control | provider-pattern facade | prompt/version registry, eval logs |

> Q1, Q2, Q4 are intentionally separated: for the entry scanner (Q2) and IoT ingestion (Q4) the primary characteristic is **operating offline under patchy Wi-Fi**, which Ticketing does not have (Q1, where payment consistency matters). A classic technique of "different characteristics → different quanta."

**Focus (MVP vs roadmap):**
- **MVP core — 7 business capabilities uniting 10 of 13 quanta** (Q1–Q9 + Q13; Q5 starts inside Q4, Q9 inside Q6, a separate split in R2) + AIP + sensor layers (Edge AI Platform, IoT, Mobile App, RFID). *(Q13 — MVP minimum: registry + inspections.)* Grouping of capabilities is in [mvp-vs-roadmap.md](mvp-vs-roadmap.md).
- **Roadmap:** Q11 Social Marketing (R1) · Q12 Ops&Scheduling (R1) · Lakehouse+Feature Store (R1) · splitting off Q5/Q9 (R2) · full agentic layer (R2) · Q10 Transport/autonomy (R3).
Details and split signals are in [mvp-vs-roadmap.md](mvp-vs-roadmap.md).

Sensor layers (not quanta): **Edge AI Platform** (a single edge runtime for fixed RGB/thermal cameras: Camera Processing · Animal Detection · Crowd Analytics · Queue Monitoring · Local Buffering; pipelines by node configuration, managed via the AIP), **IoT sensors (MQTT)**.

Client channels: **Visitor Web/Mobile App (PWA)** — visitors (profile/bookings/route/preferences/notifications/AI chat/purchases); **RFID token** (wristband/pass) — an offline complement to the app for entry/cashless/location/family linkage (ADR-027); **Staff Portal** — staff/veterinarians/management (host for Operations/Animal/Management agents).

The diagram of quanta and flows is in `../03-views-and-perspectives/` (+ export in `../diagrams/`).

## Monetizing the carnivorous plant collection (assumption)

Brief §B/§C names the carnivorous plant collection an asset that will have to be **sold** if the estate does not become profitable. There is no explicit requirement to monetize it in §D — therefore this is an **(assumption)**. It is implemented **without a new quantum**, as another *type of exhibit* on top of the existing ones:

| Earning/retention mechanic | Reused quantum |
|---|---|
| **Carnivorous plant feeding shows:** CV detects the "caught prey" moment → scheduled demonstration feedings | Fixed Cameras + **Q12** (feeding schedule) + **Q7** (announcement "feeding at 15:00") |
| **Viral content:** highlight-mining of the capture moment → auto-clip to social media | **Q11** |
| **Plant zone in the route quest** + commercial weight | **Q6** + **Q1** |
| **Zone popularity** (payback, where to invest) | **Q3** |
| **Collection health:** multispectral/CV condition assessment → preserve the asset | **Q5**-like pipeline |
| **In-app care guide:** cameras + IoT sensors (light, humidity, t°, watering) collect data on conditions and condition → AI produces quality-care recommendations → delivered in the **mobile app**: to visitors — educational content, to staff/gardeners — operational hints | Fixed Cameras + **Q4** (IoT) + **Q5**-like + **Mobile App (PWA)** + AIP |
| **Direct revenue:** premium tour / seedling mini-shop | **Q1** |

Bottom line: the plant collection becomes a **source of revenue and content**, not an asset "up for sale," at the cost of merely configuring existing quanta (with no new components).
