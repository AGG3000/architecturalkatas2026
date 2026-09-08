# MVP (7 capabilities) vs Roadmap

We designed **13 quanta (Q1–Q13) + AIP**, but for the MVP we ship a **complete core** — **7 business capabilities = 10 of 13 quanta** (roadmap = Q11–Q12 and **Q10 autonomy**; the **manned electric transport + data-mule (the Q10 asset) are MVP infrastructure**) — with a splitting roadmap, rather than 13 partially developed quanta.

## Phasing at a glance

What we do **now** (the key core) and what we **take later** — by phase and layer. The rationale and split signals are in the tables below.

| Phase | Business quanta | AI capabilities | Platform |
|---|---|---|---|
| **MVP (now)** | Q1+Q2 · Q3 · Q4+Q5 · Q6+Q9 · Q7 · Q8 · Q13 (min.) | Crowd Prediction · Dynamic Pricing (corridors) · Visitor/Operations RAG | AI Gateway (basic) · edge store-and-forward · fixed Wi-Fi/mesh + cellular uplink (dual-carrier) · sensors · manned electric land-train (Q10 asset) + data-mule |
| **R1** | Q11 Social Marketing · Q12 Ops & Scheduling | AI social marketing · AI scheduling | Lakehouse + Feature Store + Semantic Layer · governance/experimentation platform |
| **R2** | split Q5 (from Q4) · Q9 (from Q6) | full agentic layer (Animal/Management) · Q13 predictive maintenance | two-tier agent memory |
| **R3** | Q10 autonomy | autopilot (supervised→driverless) · nightly driverless data-mule sweep (nocturnal telemetry) | autonomy removes the driver OpEx — the manned transport + daytime data-mule already ship in the MVP |

**Notations:** `Q1+Q2` — start as a single deployment unit; `Q13 (min.)` — minimal in the MVP (registry + inspection rules), the "smart" part (predictive maintenance) comes later; **"split Q5 (from Q4)"** — the quantum gets its own DB/deployment on a load signal. Each shift to the right has a trigger signal (see the Roadmap table below).

**Data delivery in the MVP:** fixed Wi-Fi/mesh where cost-effective, + **directional radio bridges (PtP/PtMP)** for stationary remote points with line-of-sight (real-time without monthly cellular, available from day 1) + **cellular uplink** (dual-carrier/satellite) where there is no LoS, on top of **edge store-and-forward** buffering (it handles patchy Wi-Fi on its own). **The data mule is not a condition for the MVP to work but a cost optimization**: it rides the **manned electric transport from the MVP** (the same daytime passenger runs), removing some cellular gateways at the remote enclosures among the 55 — MVP connectivity does not depend on it. See [ADR-004](../04-adrs/ADR-004-edge-connectivity-mesh-cellular-data-mule.md), [ADR-017](../04-adrs/ADR-017-connectivity-failure-degraded-operation.md), [cost.md](../03-views-and-perspectives/cost.md).

## MVP — 7 business capabilities (10 of 13 quanta) + AIP
| MVP capability | Absorbs quanta | Why together |
|---|---|---|
| Ticketing & Access | Q1 (+ dynamic pricing) + Q2 | Q2 is a thin edge client of ticketing |
| Animal Monitoring & Health | Q4 + Q5 | one animal domain; ingestion + inference |
| Visitor Experience | Q6 (+ route quest, RAG concierge) + Q9 | customer-facing engagement, low-SLA |
| Analytics | Q3 (+ Crowd Prediction) | read-side + queue forecast |
| Ride & Attractions | Q13 | status/availability/safety inspections of 40 rides (MVP minimum: registry + rules) |
| Notifications | Q7 | delivery |
| Identity | Q8 | buy |
| AI Platform / Gateway | AIP | cross-cutting; RAG assistant as a capability behind the AIP |
| *(sensor layers)* | Fixed Cameras, IoT/MQTT, Mobile App | needed in the MVP but not business quanta |

**AI capabilities in the MVP:** Crowd Prediction (in Q3), Dynamic Pricing (in Q1, basic corridors), Visitor/Operations RAG assistants (behind the AIP). **In the roadmap:** a full agentic layer (Animal/Management agents), Lakehouse+Feature Store+Semantic Layer, predictive maintenance (Q13).

**Necessarily separate from the start:** Ticketing (PCI) · edge/offline · Animal (throughput/accuracy) · Analytics (read-side).

## Roadmap (split on signal)
| Phase | We split off | Signal |
|---|---|---|
| R1 | Q11 Social Marketing | demand for growth via social media, a content team exists |
| R1 | Q12 Operations & Scheduling | manual planning cannot handle the scale |
| R2 | Q5 from Q4 | inference load interferes with ingestion |
| R2 | Q9 from Q6 | feedback volume requires its own DB |
| R3 | Q10 autonomy (autopilot/driverless) | supervised-driving stats + budget + regulatory |
| R3 | **Nightly driverless trips for metrics** (first autonomy phase — no crowds/children) | supervised-driving statistics accumulated; ADR-026 |
| R1/R2 | **AI Governance / experimentation platform** (A/B testing, full Prompt Registry, Experiment Tracking, data-driven Model Router) | growth in the number of AI scenarios and load |
| R1 | **Lakehouse + Feature Store + Semantic Layer** | unified features/analytics needed for AI domains at scale |
| R2 | **Full agentic layer** (Animal/Management agents + two-tier memory) | value of RAG assistants proven, the number of scenarios grows |
| R2 | **Q13 predictive maintenance** (AI) | history of ride cycles accumulated (cold-start) |

> **AI Gateway (AIP) at MVP:** Prompt Adapter + Evaluation Engine + provider abstraction/fallback + basic guardrails/cost/PII-redaction/audit + lightweight prompt versioning — they immediately reduce vendor lock-in and quality degradation (see ADR-008). The full governance/experimentation platform is roadmap.

**Principle:** modular monoliths along boundaries → split into services when a force (load/team/evolution) requires it. Granularity is a consequence of forces, not an end in itself.
