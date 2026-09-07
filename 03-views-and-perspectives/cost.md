# Cost analysis (monthly, estimates)

Order of magnitude (cloud AWS/GCP class), not a final budget. **Three growth scenarios:** **MIN** (~5000/day, MVP scope) · **PROJECTED** (~15000/day, 3-year target, full roadmap) · **RAPID** (~25000/day, accelerated growth). Estate context: operational footprint ~100 acres (~40 ha), 55 enclosures, 40 rides.

## OpEx (per month)
| Line item | MIN (5k/day) | PROJECTED (15k/day) | RAPID (25k/day) |
|---|---|---|---|
| Compute (MVP: 7 capabilities / 10 quanta; later — split Q1–Q13) | ~$400 | ~$1,400 | ~$2,400 |
| Event backbone (managed stream) | ~$150 | ~$500 | ~$800 |
| Operational DBs (per-quantum) | ~$300 | ~$900 | ~$1,500 |
| **Lakehouse + Feature Store + Semantic Layer** (ADR-022; MIN — minimal warehouse) | ~$150 | ~$600 | ~$1,000 |
| Object storage + CDN (360, media, content) | ~$80 | ~$300 | ~$500 |
| **AI Gateway (AIP): LLM/CV inference + cache + router/eval** | ~$220 | ~$750 | ~$1,300 |
| **Agent platform (orchestration, multi-step tool-calling)** | ~$80 | ~$300 | ~$550 |
| **RAG (vector DB + embeddings)** | ~$60 | ~$200 | ~$350 |
| Edge-CV: model training/retraining (cloud) | ~$100 | ~$250 | ~$400 |
| **Edge ownership for cameras** (24/7 power · depreciation/refresh of boxes · MLOps/OTA to the device fleet) | ~$220 | ~$350 | ~$500 |
| **Connectivity: dual-carrier SIM + satellite (ADR-017)** | ~$80 | ~$200 | ~$350 |
| IdP · maps · observability | ~$200 | ~$600 | ~$1,000 |
| **Shuttle fleet lease** (Q10 — roadmap: none in MIN; PROJECTED ~5–6 units; RAPID more) | — | ~$3,000 | ~$5,000 |
| **Total OpEx** | **≈ $2,040/mo** | **≈ $9,350/mo** | **≈ $15,650/mo** |
| PSP fees | pass-through (~2.9%+$0.30/txn) | pass-through | pass-through |

> Q13 (Ride & Attractions) is included in the compute line (MVP minimum — registry + rules; predictive maintenance — roadmap). The shuttle lease is counted in OpEx as a roadmap phase (R3).

> **Cost-per-task (agent sessions).** One completed agent session ≈ Model Router + ~3–6 LLM calls + RAG embeddings + tool inference. With a default cheap model + cache + **step limit**, the reference figure is **~$0.01–0.03 / session**. Agent tokens are in the **AI Gateway (AIP)** line, orchestration — in **Agent platform**. At ~2000 agent sessions/day (PROJECTED) ≈ **$20–60/day** — already counted in these two lines; a **token budget and a per-session step limit** keep the cost from token-blow-up on long chains (see [agents.md](../05-ai/agents.md)).

## CapEx (one-time, MVP; without the autonomous shuttle)
MQTT climate/breach/RFID sensors (~40, air/water climate per enclosure — cheap, $10–50 each) + **MQTT sensors for ride mechanisms (~40: vibration/temperature/current for predictive maintenance, budget §E)** + RGB (**~15 in the MVP — by enclosure clusters + priority queue/plant zones; overview/PTZ, not 1:1 per enclosure**) + thermal (~4, only fire/overheating of rides/intruders/warm-blooded animals) + gate controllers (~6) + **Edge AI Platform nodes (~4–6 × ~$800 ≈ $4–5k; a single runtime, instances by camera cluster, not a "box per zone")** + mesh + **directional radio bridges (PtP/PtMP, ~4–8 links × ~$150–400) for stationary remote zones/edge nodes with LoS** ≈ **$19,000**.
*(Radio bridges — one-time CapEx, but they remove part of the monthly cellular OpEx at stationary remote points; see ADR-004.)*
*(Camera coverage is phased: the MVP covers priority zones and enclosure clusters, while cheap MQTT climate/breach sensors sit on **every** enclosure (they do not depend on cameras); extending RGB to all 55 enclosures/zones — as growth proceeds, the Compute/Edge line scales in PROJECTED/RAPID.)*
*(Edge ownership is not only CapEx: power, depreciation/refresh roughly every ~3 years and MLOps/OTA are counted in the OpEx line "Edge ownership for cameras.")*
The autonomous shuttle is **leased** (roadmap R3), not CapEx. Fleet: MVP ~3 units (2 active + 1 spare charging), scale ~5–6 (2 routes) — the size is set by passenger demand; the same fleet covers the data mule without separate vehicles.

## Growth by scenario and control levers
- **Total by scenario:** MIN ≈ **$2.0k/mo** · PROJECTED ≈ **$9.4k/mo** · RAPID ≈ **$15.7k/mo**.
- **Costs grow slower than traffic.** Traffic MIN→PROJECTED grows 3× (5k→15k visits/day), while costs grow less: edge-CV, IdP and RAG barely depend on the number of visitors (the cameras and sensors are the same, and the RAG assistant does not care how many people ask). This means **cost per visitor falls** with growth.
- **The $2.0k→$9.4k jump is new capabilities, not the same thing getting more expensive:** PROJECTED includes roadmap components that the MVP lacks — the **shuttle lease** (~$3k/mo) and the **full Lakehouse** (a minimal warehouse in the MVP).
- **What makes the solution cheaper (consequences of the design):**
  - **edge-CV** — recognition on site, raw video does not go to the cloud → no charge for outbound traffic (this would be the main cost with cloud CV);
  - **data mule** — data is carried by the shuttle → no need for a paid gateway at each of the 55 enclosures;
  - **serverless scale-to-zero** — a service consumes no resource while idle → we pay only for usage;
  - **AI Gateway cost-monitor** — the expensive model is called only when necessary, the default is the cheap one;
  - **selective thermal** — expensive thermal cameras (~$800) are placed pointwise, not everywhere.
- **Availability: we target 99.9–99.99% (3–4 "nines"), not 99.999%.** 99.9% ≈ 8.8 h downtime/year, 99.99% ≈ 52 min/year, 99.999% ≈ 5 min/year — but each extra "nine" costs many times more (full redundancy, multi-region). For the park, 3–4 "nines" is enough — **a deliberate "availability ↔ cost" trade-off** under a limited budget.

## Conclusion
An affordability estimate *(the ticket price is not set in the brief — **assumption**)*: at a target of **15,000/day ≈ 450,000 visits/mo** and an average admission price of **~$20–30**, ticket revenue ≈ **$9–13M/mo**. Cloud IT PROJECTED (**≈ $9.4k/mo**) is **~0.1% of revenue**. Conclusion: **IT/cloud is not a budget constraint** — the estate's main expenses lie outside IT (animal care, veterinary services, staff, fleet lease). Even at a conservative price (~$10), IT stays <0.3% of revenue.
