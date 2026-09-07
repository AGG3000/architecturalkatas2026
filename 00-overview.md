# Overview — how AI solves the estate's problems

A short narrative (≤2 screens). We designed a new comprehensive architecture for the estate of the 72nd Countess von Digitalis, where AI is not a showpiece but a working tool, embedded into an event-driven architecture of independent quanta and reliably governed under constraints (patchy Wi-Fi, budget, safety, reputation).

## The Countess's problems (from the brief)
- It's unclear which parts of the estate are popular → hard to invest and to allocate staff.
- Caring for 200+ animals is expensive, especially when they fall ill → healthy animals are needed (+ piranha population control).
- Visitors must be grown (5000 → 15000/day) along with profitability, and returning guests attracted.
- All of this under **patchy Wi-Fi** and with a budget for **MQTT devices**.

## Business value of AI
**What is the most "expensive" thing in the solution.** The brief's existential goal is **profitability** ("otherwise, back to the gnomes"). The main AI lever is **not** animal monitoring, but **revenue**:

> **Profit = Attendance × Spend-per-guest × Repeat-rate** — and AI hits all three factors (all on already-existing features).

| Revenue lever | AI features |
|---|---|
| **Attendance ↑** | AI Social Marketing (highlight-mining → viral content, Q11) · Dynamic Pricing (off-peak discounts → accessibility + smoothing, Q1) · return forecast (Q6) |
| **Spend per guest ↑** | AI Concierge (the route leads to commercial zones/shows, Q6) · Queue Prediction (less time in queues → more time for food/souvenirs, Q3) · Dynamic Pricing (yield) |
| **Repeat rate ↑** | return / retention forecast (Q6) + personalization on **RFID** data (ADR-027) + Concierge UX |

**Flywheel:** RFID data → personalization → Concierge / retention → more visits and spend → even more data.

**Why this is the "most expensive" feature:** attendance runs into capacity (crowds near venomous animals and 18th-century attractions are limited), so **spend-per-guest + repeat is the most capital-efficient lever** (it grows without growing the crowd).

**Animal AI — a foundation, not a revenue engine:** healthy animals = *the product itself*, protection of an expensive asset and prevention of reputational-financial losses (death of a rare/venomous animal). A necessary foundation (cost-avoidance / enabler), but revenue is driven by the revenue loop above.

## How AI solves this (the core)
- **Animal health (Q5) — the foundation (product + asset, not a revenue engine):** RGB-CV (behavior/feeding/re-ID) + underwater RGB (piranhas) + breach sensors/RFID (escape/accounting of cold-blooded animals) + **MQTT climate sensors** (air/water microclimate) + anomaly detection on telemetry → early alert to the keeper. Thermal imaging — pointwise (warm-blooded/night/fire/intruders); it does not see cold-blooded animals (see ADR-011).
- **Popularity analytics (Q3):** edge-CV on cameras measures queues/density → zone heatmap + NL queries over the data for the Countess ("where is it under-loaded on Tuesday?").
- **Growth and return (Q6):** a dynamic route-quest (minimum queues × experiences × commerce) + AI generation of quests; return forecast.
- **Profitability:** AI social marketing (Q11) from vivid camera moments; AI schedules for staff/feeding (Q12); autonomous shuttle (Q10).
- **Plant collection (assumption):** cameras + IoT sensors → feeding shows, viral content, and a care guide in the app — an asset "for sale" becomes a source of income (quantum reuse).

## How AI is embedded reliably
- **Uncertainty:** model provider abstraction, fallback, cost monitoring (`05-ai/uncertainty.md`).
- **Validation:** human-in-the-loop for critical cases, fitness functions, monitoring in production (`05-ai/validation-verification.md`).
- **Constraints:** edge inference + data mule under patchy Wi-Fi (`03-views-and-perspectives/deployment.md`).
- **AI Governance:** risk classification of use cases, guardrails, bias/fairness, transparency, model lifecycle, and AI incident response (`05-ai/governance.md`).
- **Unified state view:** the park's "digital twin" — domain projections/aggregates on top of the Lakehouse (not a separate component), giving AI agents holistic context for decisions.

## Key decisions
A single list — **[Key Architectural Decisions](README.md#key-architectural-decisions)** (10 key ones) + the full [ADR index](04-adrs/README.md). Briefly, grouped by what they address:

- **Style for heterogeneous characteristics:** event-driven backbone + architectural quanta, starting as modular monoliths, evolutionary split-off (**ADR-001**).
- **Suitability under patchy Wi-Fi:** edge inference + mesh/cellular/**data-mule** connectivity (**ADR-004**).
- **AI uncertainty:** provider abstraction + fallback + cost-monitor on the **AI Gateway** (**ADR-008**).
- **AI validation:** human-in-the-loop + confidence thresholds for critical cases (**ADR-010**), on top of it — the **AI Governance** framework (risk classification, guardrails, bias/fairness, model lifecycle, incident) (**ADR-018**).
- **AI use cases (innovative + profitability):** **RAG assistants** with grounding/citations (Visitor Concierge + Operations Copilot, **ADR-019**), **Crowd Prediction** — forecasting queues ahead for staff allocation (**ADR-020**), **Dynamic Pricing** within corridors (**ADR-021**).
- **AI embedded in business processes:** an **agentic layer** by stakeholder on typed tools + HITL (**ADR-023**) on top of the unified data/ML platform **Lakehouse + Feature Store + Semantic Layer** (**ADR-022**).

*Infrastructure depth (edge MQTT store-and-forward **ADR-003/007**, "rules→AI" cold-start **ADR-009**, edge-CV on cameras **ADR-011**, autonomous shuttle **ADR-015**, privacy/PCI **ADR-006/013**) — in the [ADR index](04-adrs/README.md).*
