# AI — map of applications

The kata theme is "AI-Assisted." All AI calls go through the cross-cutting **AI Gateway (AIP)** to a pool of providers (GPT/Claude/Mistral · CV · autonomy). Inside: **Prompt Adapter · Model Router (policy-based) · Evaluation Engine · Prompt Registry** + cross-cutting concerns (provider abstraction/fallback, guardrails, PII redaction on egress, rate-limit, cost-monitor/cache, audit, keys). Internals — [diagrams/ai-gateway.md](../diagrams/ai-gateway.md).

> **Business value (in one frame):** the main AI lever is **revenue**, not animal monitoring. **Profit = Attendance × Spend-per-guest × Repeat-rate**: attendance ↑ (Marketing/Pricing/return-prediction) · spend ↑ (Concierge/Queue Prediction/Pricing) · repeat ↑ (retention + RFID personalization). **Animal AI is the foundation** (product + asset), not the revenue engine. Expanded — [00-overview → Business value of AI](../00-overview.md#business-value-of-ai).

| AI application | Domain | Brief pain point |
|---|---|---|
| CV animal monitoring (RGB: behavior/feeding/re-ID; thermal: reptile microclimate/warm-blooded/night; cold-blooded — RGB+breach/RFID) | Q5 | healthy animals |
| Piranha population counting (CV) | Q5 | population control |
| Health anomaly detection from telemetry | Q5 | early alert, cheaper than treatment |
| Queues/density/heatmap (edge-CV cameras) | Q3 | where to invest/staff |
| NL analytics for the Countess (text-to-query) | Q3 | understanding popularity |
| Investment ROI assessment (before/after with seasonality adjustment) | Q3 | where to invest — causally, not just heatmap |
| Dynamic route-quest + AI quest generation | Q6 | growth, engagement, return |
| Return prediction | Q6 | profitability |
| **Crowd Prediction:** forecasting queues/density ahead (ML on Q3 history + events/weather) | Q3 + AIP → Q6/Q12/Q7 | proactive route, staff in advance, shorter queues |
| **Dynamic Pricing:** dynamic ticket/membership prices (demand/load/time, within corridors) | Q1 + Q6 + AIP | profitability + peak smoothing |
| AI social marketing: highlight-mining from cameras + post generation | Q11 | growth, return |
| AI staff/feeding scheduling (constraint-opt) | Q12 | profitability, welfare |
| Autonomous shuttle driving (perception/planning) | Q10 | costs, 24/7 |
| Feedback sentiment | Q9 | what to improve |
| CV detection of "plant caught prey" → feeding show + content *(assumption)* | Fixed Cameras + Q12/Q7/Q11 | monetizing the plant collection |
| Plant care guide: cameras + IoT sensors → AI recommendations in the mobile app *(assumption)* | Fixed Cameras + Q4 + Q5-like + Mobile App | quality care + educational content |
| RAG assistant: staff guide to protocols/SOPs + visitor support (grounded, with citations) *(assumption)* | AIP + Knowledge Base + Mobile App | animal/plant care + support/UX (recommendations remain ML in Q6) |

**Reuse:** a single camera edge-CV serves welfare (Q5), marketing (Q11), and analytics (Q3) alike.

## AI portfolio: 5 core capabilities + additional

Two tiers: the five core capabilities close the brief's direct pain points; the second tier shows the breadth of AI application (without bloating the flagship set).

### Tier 1 — Core capabilities (brief pain points)

| Capability | What it does | Implementation |
|---|---|---|
| **Visitor Concierge Agent** | route planning + chat assistant | Q6 route-quest + RAG visitor assistant ([ADR-019](../04-adrs/ADR-019-rag-knowledge-assistant.md)) |
| **Animal Health AI** | animal telemetry analysis | Q5 (anomaly detection + CV) |
| **Crowd Prediction AI** | forecasting queues ahead | Q3 + AIP ([ADR-020](../04-adrs/ADR-020-crowd-prediction-ai.md)) |
| **Dynamic Pricing AI** | ticket/membership pricing | Q1 + Q6 ([ADR-021](../04-adrs/ADR-021-dynamic-pricing-ai.md)) |
| **Operations Copilot** | assistance for park staff | staff RAG assistant ([ADR-019](../04-adrs/ADR-019-rag-knowledge-assistant.md)) + operational hints |

### Tier 2 — Additional AI applications (breadth of application)

| Application | What it does | Implementation |
|---|---|---|
| **AI Social Marketing** | highlight-mining of standout moments from cameras → auto-generation of posts | Q11 + AIP ([ADR-016](../04-adrs/ADR-016-ai-marketing-and-scheduling-hitl-provider-abstraction.md)) |
| **AI Scheduling** | constraint optimization of shifts/feeding/maintenance (welfare × crowd) | Q12 + solver/AIP ([ADR-016](../04-adrs/ADR-016-ai-marketing-and-scheduling-hitl-provider-abstraction.md)) |
| **Plant monetization** *(assumption)* | CV detection of "plant caught prey" → feeding show + viral content; care guide | Fixed Cameras + Q12/Q7/Q11 + Mobile App |
| **Autonomous shuttle + data-mule** | autonomy (supervised→driverless) + data delivery under patchy Wi-Fi (DTN) | Q10 ([ADR-015](../04-adrs/ADR-015-autonomous-shuttle-and-data-mule.md)) |
| **Feedback Sentiment** | feedback sentiment → what to improve | Q9 + AIP |

*The second tier is genuinely worked out (its own ADRs/quanta/costs/risks), but deliberately kept outside the "top five" to maintain focus. All applications are in the table above and in the [ADR index](../04-adrs/README.md).*

## AI taxonomy (class → how we validate)
We distinguish **three classes of AI** and treat them differently (this dictates V&V):
- **Classical ML** (forecast/prices) — deterministic given data → **tested as software** (metrics + CI gate).
- **CV** (vision) — probabilistic → **confidence thresholds + human** in the loop.
- **Generative** (LLM) — non-deterministic → **grounding + guardrails + continuous eval**.
*(An agent = an orchestrator on top of these classes via typed tools.)*

**"AI only where needed" (why it is not a rule).** Every capability passed the filter "why is deterministic logic insufficient": Animal Health — anomalies are individual, not expressible by a fixed threshold; Crowd Prediction — a rule doesn't catch seasonality/event effects; Concierge — open-ended NL dialogue; Dynamic Pricing — demand elasticity; Predictive Maintenance — degradation in multi-signal patterns. Where a rule suffices — **we don't introduce AI** (current zone popularity = counters + dashboard; ticket validation = signature, not a model).

## End-to-end traceability: goal → OKR → capability → AI class → implementation → fallback
Every AI capability is **tied to a number it moves**, and has a **deterministic fallback** (AI is optional on the critical path — on failure the system degrades rather than crashes). OKR figures are an **assumption/calibration** (the brief doesn't set them), except 15,000/day (fact §C).

| Business goal | OKR (target) | Capability | AI class | Implementation | Deterministic fallback |
|---|---|---|---|---|---|
| Growth of repeat visits | +20% repeat rate *(assum.)* | Personalization / Concierge | Predictive + Generative | Visitor Agent — Q6 + RAG ([ADR-019](../04-adrs/ADR-019-rag-knowledge-assistant.md)/[023](../04-adrs/ADR-023-agentic-layer.md)) | static recommendation rules (popular/nearest) |
| Growth of attendance | 5000 → **15000/day** *(fact §C)* | AI social marketing + Dynamic Pricing | Generative + Predictive | Q11 + Q1 ([ADR-016](../04-adrs/ADR-016-ai-marketing-and-scheduling-hitl-provider-abstraction.md)/[021](../04-adrs/ADR-021-dynamic-pricing-ai.md)) | fixed post schedule + fixed prices |
| Profitability | AI/cloud ≤ ~1% of revenue *(assum., see [cost](../03-views-and-perspectives/cost.md))* | Dynamic Pricing | Predictive + rules | Q1/Q6 ([ADR-021](../04-adrs/ADR-021-dynamic-pricing-ai.md)) | fixed prices within min/max corridors |
| Zone popularity + staffing | −20% peak queues *(assum.)* | Crowd Prediction | Classical ML (forecast) | Q3 + AIP ([ADR-020](../04-adrs/ADR-020-crowd-prediction-ai.md)) | day-of-week/time-of-day heuristics |
| Healthy animals (↓ costs) | early-detection recall ≥ 0.95 | Animal Health | CV + anomaly | Q5 + Animal Agent ([ADR-010](../04-adrs/ADR-010-human-in-the-loop-confidence-thresholds.md)) | threshold rules on telemetry |
| Piranha population control | count error ≤ 10% | Piranha counting | CV | Q5 edge-CV | manual periodic counting |
| Ride uptime/safety | −30% unplanned downtime *(assum.)* | Predictive Maintenance | Predictive (anomaly + RUL) | Q13 ([ADR-025](../04-adrs/ADR-025-ride-attractions-management.md)) | calendar maintenance + inspection rules |
| Operational efficiency | 0 hard-constraint violations | Scheduling / Ops Copilot | Constraint-opt + Generative | Q12 + Operations Agent ([ADR-016](../04-adrs/ADR-016-ai-marketing-and-scheduling-hitl-provider-abstraction.md)/[023](../04-adrs/ADR-023-agentic-layer.md)) | manual schedules/templates |

**Invariant:** each capability is invoked as a named unit behind the AIP → resolves to a versioned bundle {model+prompt+eval} ([ADR-008](../04-adrs/ADR-008-ai-platform-provider-abstraction-fallback-cost.md)); business code does not know the model. Safety-critical paths (welfare alerts, gate access) **do not depend on generative AI**.

## AI embedded in business processes: agents by stakeholder + shared memory

AI helps **4 groups directly** via agents that rely on shared memory (Lakehouse — long-term; the fast tier — working):
- **Visitor Agent** → visitors · **Operations Agent** → staff · **Animal Agent** → veterinarians · **Management Agent** → management.
Lakehouse is the **memory and analytical layer**, not a separate warehouse with ML on the side. Agents work with the park's **"digital twin"** — a set of up-to-date domain projections/aggregates (gold + Semantic Layer) on top of the Lakehouse (not a separate component; see [data.md](../03-views-and-perspectives/data.md)). Details — [agents.md](agents.md), [ADR-023](../04-adrs/ADR-023-agentic-layer.md), [ADR-024](../04-adrs/ADR-024-lakehouse-as-agent-memory.md).

Details: [agents](agents.md) · [uncertainty](uncertainty.md) · [validation & verification](validation-verification.md) · [governance](governance.md) (risk classification, bias/fairness, transparency, model-lifecycle, incident).
