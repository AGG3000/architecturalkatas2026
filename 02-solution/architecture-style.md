# Architecture style

## Driving characteristics (top-3)
1. **Offline-tolerance** — patchy Wi-Fi, data from the estate to the cloud.
2. **Evolvability** — the AI part changes quickly (models/providers).
3. **Scalability/Elasticity** — growth 5000→15000/day, event peaks.
*(+ Security for ticketing, Accuracy for AI animal health.)*

## Style comparison worksheet (in text)
Scoring of candidates against the driving characteristics (★ = worse … ★★★★★ = better). Scores are fixed; the rationale for the choice is below the table.

| Characteristic | Monolith | Service-based | Microservices | **Event-driven + quanta** |
|---|---|---|---|---|
| Offline-tolerance (edge) | ★ | ★★ | ★★★ | **★★★★** |
| Evolvability | ★ | ★★★ | ★★★★ | **★★★★★** |
| Scalability/Elasticity | ★ | ★★★ | ★★★★★ | **★★★★★** |
| Cost/complexity (startup) | ★★★★★ | ★★★★ | ★★ | **★★★** |
| Alignment of AI characteristics | ★★ | ★★★ | ★★★★ | **★★★★★** |

**Weighting (important):** the scores are read **with priority on the top-3 drivers** (offline-tolerance · evolvability · scalability) — it is precisely on these that event-driven + quanta wins decisively; **Cost/complexity is a deliberate trade-off** (accepted and mitigated by starting with modular monoliths), so its loss to the monolith does not overturn the choice. **Accuracy and Security are quantum-level drivers** (Q5 animal health, Q1 ticketing/PII), not a choice of the system-wide style: they are addressed inside the corresponding quanta, and therefore fall outside the top-3 system characteristics.

**Rationale for key scores (why event-driven + quanta):**
- **Offline-tolerance ★★★★.** The event model (store-and-forward, at-least-once, idempotency by event id) is natural semantics for the edge under patchy Wi-Fi: the device buffers events and forwards them in batches when connectivity returns. Not ★★★★★ only because some operations (payment Q1) still require online consistency — offline is not absolute.
- **Evolvability ★★★★★.** Quanta are loosely coupled via the bus, each with its own DB and its own style; AI domains live behind a provider abstraction (AIP), so switching a model/provider does not affect neighbors — a direct answer to "AI changes quickly."
- **Scalability/Elasticity ★★★★★.** Asynchronous consumers scale independently by load (event peaks, growth 5000→15000/day); the read-side (CQRS, Q3) scales separately from operations and does not bring down transactions.
- **Cost/complexity ★★★.** An honest downside: an event bus + distribution is more expensive than a monolith to operate. We mitigate this by starting as modular monoliths along quantum boundaries and splitting into services only on a load signal — hence not ★★ (as with pure microservices), but not higher either.
- **Alignment of AI characteristics ★★★★★.** Quantum boundaries are drawn along diverging driving characteristics; AI additions (CV, LLM, autonomy) fit into their own quanta behind the AIP without blurring the characteristics of neighbors — the AI characteristics are aligned with the architecture.

## Choice + rationale
**An event-driven architecture of architectural quanta**, each with its own style along diverging driving characteristics. Quanta start as **modular monoliths** and split into services on a load/team signal (evolutionarily). Edge quanta (Gate/IoT/shuttle/cameras) are designed for offline. AI sits behind a cross-cutting provider abstraction (AIP).

The full map is in `domain-map.md`; the granularity choice is in `mvp-vs-roadmap.md`. The rationale is in ADR-001 (style) — `../04-adrs/`.
