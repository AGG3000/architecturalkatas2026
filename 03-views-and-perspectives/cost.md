# Cost analysis (monthly, estimates)

Order of magnitude (cloud AWS/GCP class), not a final budget. **Three growth scenarios:** **MIN** (≈5000/day, MVP scope) · **PROJECTED** (≈15000/day, 3-year target, full roadmap) · **RAPID** (≈25000/day, accelerated growth). Estate context: operational footprint ≈100 acres (≈40 ha), 55 enclosures, 40 rides.

## OpEx (per month)
| Line item | MIN (5k/day) | PROJECTED (15k/day) | RAPID (25k/day) |
|---|---|---|---|
| Compute (MVP: 7 capabilities / 10 quanta; later — split Q1–Q13) | ≈$400 | ≈$1,400 | ≈$2,400 |
| Event backbone (managed stream) | ≈$150 | ≈$500 | ≈$800 |
| Operational DBs (per-quantum) | ≈$300 | ≈$900 | ≈$1,500 |
| **Lakehouse + Feature Store + Semantic Layer** (ADR-022; MIN — minimal warehouse) | ≈$150 | ≈$600 | ≈$1,000 |
| Object storage + CDN (360, media, content) | ≈$80 | ≈$300 | ≈$500 |
| **AI Gateway (AIP): LLM/CV inference + cache + router/eval** | ≈$220 | ≈$750 | ≈$1,300 |
| **Agent platform (orchestration, multi-step tool-calling)** | ≈$80 | ≈$300 | ≈$550 |
| **RAG (vector DB + embeddings)** | ≈$60 | ≈$200 | ≈$350 |
| Edge-CV: model training/retraining (cloud) | ≈$100 | ≈$250 | ≈$400 |
| **Edge ownership for cameras** (24/7 power · depreciation/refresh of boxes · MLOps/OTA to the device fleet) | ≈$220 | ≈$350 | ≈$500 |
| **Connectivity: site fixed uplink + dual-carrier SIM + satellite (ADR-017)** | ≈$80 | ≈$200 | ≈$350 |
| IdP · maps · observability | ≈$200 | ≈$600 | ≈$1,000 |
| **Transport fleet lease** (Q10 — manned electric from the MVP; PROJECTED ≈5–6 units + autonomy; RAPID more) | ≈$1,500 | ≈$3,000 | ≈$5,000 |
| **Total OpEx** | **≈ $3,540/mo** | **≈ $9,350/mo** | **≈ $15,650/mo** |
| PSP fees | pass-through (≈2.9%+$0.30/txn) | pass-through | pass-through |

> Q13 (Ride & Attractions) is included in the compute line (MVP minimum — registry + rules; predictive maintenance — roadmap). The transport lease appears in OpEx **from the MVP** (manned electric, ~3 small units); **autonomy** is the R3 phase.

> **Internet & data transfer.** The mobile (cellular) edge→cloud channel is in the "Connectivity" line (dual-carrier SIM), together with the site's main fixed uplink and satellite backup. **Cloud egress/data-transfer** has no separate large line **by design**: edge-CV does not push raw video to the cloud (only events/metadata leave), so outbound traffic is small and folded into the Compute/Object-storage lines.

> **Storage.** Raw stays on the edge → the analytical lake is only ~50 GB/yr (≈ $2–4/mo of storage; the Lakehouse line is mostly compute/query, not bytes). Logs/metrics/**agent traces** (sampled) + audit are a separate observability layer (≈ $50–150/mo, inside "IdP · maps · observability"). Sizing — [data.md](data.md#storage-sizing).

> **Cost-per-task (agent sessions), derived.** Per session: `cost ≈ N_calls × tokens/call × price × (1 − cache-hit) + embeddings + tool inference`.
> - **Mean:** ≈4 calls × ≈1.1k tokens × ≈$0.5/1M (cheap default) × (1 − 0.3 cache) ≈ **$0.002**. **P90:** ≈8 calls (step limit) × ≈1.5k tokens, escalated model, low cache ≈ **$0.03**. Band ≈ **$0.002–0.03 / session**.
> - **Attach-rate sensitivity** (sessions/day = 15k visitors × Concierge attach):
>
> | Concierge attach | sessions/day | ≈ cost/day | ≈ /mo |
> |---|---|---|---|
> | 13% | ≈2,000 | ≈$20 | ≈$0.6k |
> | 40% | ≈6,000 | ≈$120 | ≈$3.6k |
> | 60% (peak) | ≈9,000 | ≈$220 | ≈$6.6k |
>
> **Honest note:** the OpEx AI Gateway + Agent lines are sized at **moderate (~13%) attach**; at higher attach they scale roughly proportionally. The **step limit + cheap-default model + inference cache** bound per-session cost, and the **cost-monitor cap** (AI spend ≤ target % of revenue, [ADR-008](../04-adrs/ADR-008-ai-platform-provider-abstraction-fallback-cost.md)) throttles/degrades to fallback before the budget blows. Even ≈$6.6k/mo at 60% attach stays **≪ 0.1% of revenue** (see Conclusion) — high attach is a *good* problem (retention working). See [agents.md](../05-ai/agents.md).

## CapEx (one-time, MVP; excludes the transport fleet (leased))

| Item | Qty | ≈Unit cost | Note |
|---|---|---|---|
| MQTT climate/breach/RFID sensors | ≈40 | $10–50 each | air/water climate per enclosure (cheap) |
| MQTT ride-mechanism sensors (predictive maintenance) | ≈40 | low | vibration/temperature/current (budget §E) |
| RGB cameras | ≈15 | — | by enclosure clusters + priority queue/plant zones; overview/PTZ, not 1:1 per enclosure |
| Thermal cameras | ≈4 | ≈$800 each | fire / ride overheating / intruders / warm-blooded only |
| Gate controllers | ≈6 | — | offline ticket validation |
| Edge AI Platform nodes | ≈4–6 | ≈$800 each (≈ $4–5k) | single runtime, one instance per camera cluster (not a "box per zone") |
| WiFi mesh | — | — | coverage along routes |
| Directional radio bridges (PtP/PtMP) | ≈4–8 links | $150–400 each | stationary remote zones/edge nodes with line-of-sight |
| **Total** | | **≈ $19,000** | MVP, excl. transport fleet (leased) |

**Notes:**
- **Radio bridges** are one-time CapEx but remove part of the monthly cellular OpEx at stationary remote points (see [ADR-004](../04-adrs/ADR-004-edge-connectivity-mesh-cellular-data-mule.md)).
- **Camera coverage is phased:** the MVP covers priority zones and enclosure clusters; cheap MQTT climate/breach sensors sit on **every** enclosure (independent of cameras); extending RGB to all 55 enclosures/zones happens as growth proceeds (the Compute/Edge line scales in PROJECTED/RAPID).
- **Edge ownership ≠ CapEx only:** power, depreciation/refresh (≈every 3 years) and MLOps/OTA are counted in the OpEx line "Edge ownership for cameras."
- **The transport fleet is leased** (manned electric **from the MVP**; autonomy — R3), not CapEx. Fleet: MVP ≈3 units (2 active + 1 spare charging), scaling to ≈5–6 (2 routes) sized by passenger demand; the same fleet covers the data-mule without separate vehicles.

## Growth by scenario and control levers
- **Total by scenario:** MIN ≈ **$3.5k/mo** · PROJECTED ≈ **$9.4k/mo** · RAPID ≈ **$15.7k/mo**.
- **Costs grow slower than traffic.** Traffic MIN→PROJECTED grows 3× (5k→15k visits/day), while costs grow less: edge-CV, IdP and RAG barely depend on the number of visitors (the cameras and sensors are the same, and the RAG assistant does not care how many people ask). This means **cost per visitor falls** with growth.
- **The $3.5k→$9.4k jump is fleet scaling + autonomy + the full Lakehouse** (not the same thing getting more expensive): the MVP already runs a small manned electric fleet (≈$1.5k/mo); PROJECTED scales it and adds autonomy (≈$3k/mo total) plus the **full Lakehouse** (a minimal warehouse in the MVP).
- **What makes the solution cheaper (consequences of the design):**
  - **edge-CV** — recognition on site, raw video does not go to the cloud → no charge for outbound traffic (this would be the main cost with cloud CV);
  - **data mule** — data is carried by the shuttle → no need for a paid gateway at each of the 55 enclosures;
  - **serverless scale-to-zero** — a service consumes no resource while idle → we pay only for usage;
  - **AI Gateway cost-monitor** — the expensive model is called only when necessary, the default is the cheap one;
  - **selective thermal** — expensive thermal cameras (≈$800) are placed pointwise, not everywhere.
- **Availability: we target 99.9–99.99% (3–4 "nines"), not 99.999%.** 99.9% ≈ 8.8 h downtime/year, 99.99% ≈ 52 min/year, 99.999% ≈ 5 min/year — but each extra "nine" costs many times more (full redundancy, multi-region). For the park, 3–4 "nines" is enough — **a deliberate "availability ↔ cost" trade-off** under a limited budget.

## AI funding gate & per-capability payback (assumption)
Every AI capability must **move a number and beat its own fallback**, or it is cut — an economics gate, not a committee.

| Capability | Target effect (illustrative) | Kill-criterion — cut if… |
|---|---|---|
| Crowd Prediction | fewer peak queues / better staffing | doesn't beat the day-of-week heuristic (MAPE) within 2 seasons |
| Dynamic Pricing | revenue uplift + peak smoothing | no revenue uplift vs fixed corridors within 2 quarters |
| Concierge / retention | +repeat visits, +spend per guest | no return-rate uplift vs control (A/B) |
| Animal Health | earlier detection, lower care cost | recall < target, or vet-override rate too high |
| Predictive Maintenance | less unplanned ride downtime | no downtime reduction vs calendar maintenance |
| AI Social Marketing | attendance from content | engagement→visits below threshold |

Guardrails: total AI spend is capped (cost-monitor, [ADR-008](../04-adrs/ADR-008-ai-platform-provider-abstraction-fallback-cost.md)); a kill-criterion that never fires in ~2 years means the capability was decoration — we say so and cut it. Per-capability fitness thresholds — [validation & verification](../05-ai/validation-verification.md).

## Conclusion
An affordability estimate *(the ticket price is not set in the brief — **assumption**)*: at a target of **15,000/day ≈ 450,000 visits/mo** and an average admission price of **≈$20–30**, ticket revenue ≈ **$9–13M/mo**. Cloud IT PROJECTED (**≈ $9.4k/mo**) is **≈0.1% of revenue**. Conclusion: **IT/cloud is not a budget constraint** — the estate's main expenses lie outside IT (animal care, veterinary services, staff, fleet lease). Even at a conservative price (≈$10), IT stays <0.3% of revenue.
