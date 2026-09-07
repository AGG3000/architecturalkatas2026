# Decision log & negative space

The key cross-cutting decisions — and, just as important, **what we deliberately did *not* build.**

## Key decisions (why)
| # | Decision | Why | Rejected alternative |
|---|---|---|---|
| D1 | Event-driven **quanta**, start as modular monoliths, split by signal | diverging driving characteristics; small team on day 1 | one big monolith / microservices-from-day-1 |
| D2 | AI behind a **capability-gateway** (provider-agnostic) | provider churn / price / shutdown | direct provider calls from domains |
| D3 | **Agentic layer** (roadmap R2) on typed tools + HITL | multi-step business value; the theme's own example | autonomous agents without a human; assistants-only |
| D4 | **Deterministic fallback for every capability**; safety never on generative | AI stays optional on the critical path | AI in the critical path |
| D5 | Confidence thresholds **derived from cost-of-error** | FP/FN asymmetry (a welfare miss ≫ a false alarm) | "0.9 by convention" |
| D6 | Thermal only for warm-blooded / fire / overheating; cold-blooded via RGB + breach/RFID | physics: thermal cannot see cold-blooded animals | thermal cameras everywhere |
| D7 | Lakehouse + Feature Store + Semantic Layer via CDC (roadmap R1) | unified features/analytics without breaking quantum autonomy | shared DB / fragmented warehouses |
| D8 | Dynamic pricing by **time/load**, human-approved corridors | fairness / regulatory | per-person pricing |

## What we deliberately did NOT build (negative space)
- **Drones** — reviewed, rejected: noise/stress for venomous animals, regulatory load, cost > value; fixed cameras + the shuttle cover the need.
- **Per-person dynamic pricing** — discrimination/fairness risk; we price by time/load only.
- **Full multi-region HA in the MVP** — a deliberate availability↔cost trade-off; 3–4 nines is enough, multi-region is roadmap.
- **A self-built ML stack / training from scratch** — no data at start, unrealistic budget; we buy models behind the gateway.
- **Autonomous shuttle in the MVP** — safety/regulatory/cost; supervised→driverless only in R3 after statistics.

## The number to challenge first
The **assumed ticket price ($20–30)** carries the affordability conclusion. It is the single most load-bearing assumption — replace it with real figures before anything else.
