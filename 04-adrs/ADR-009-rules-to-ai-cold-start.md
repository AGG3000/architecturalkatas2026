# ADR-009: "Rules → AI" strategy for domains with no data at the start

## Status
Accepted

## Context
A number of AI domains have **no historical data** for training at the start: prediction of visitor return (Q6), demand forecasting and rostering (Q12), anomaly detection for animal health (Q5). There is nothing to train ML on, yet the functionality is needed from day one ("the estate is loss-making" — brief fact, monetization is needed immediately). Affected characteristics: **Evolvability, Accuracy, Cost**. The kata requirement is "dealing with uncertainty in AI" and consistency of the AI additions with the base architecture.

## Decision
At the start, domains run on **deterministic rules/heuristics** (thresholds, simple formulas, expert estimates from keepers/the manager). In parallel we accumulate labeled data through the same AI facade (AIP). Once a sufficient volume is accumulated and fitness functions are passed (accuracy on the golden-set), we switch from rules to ML **via a config switch behind the facade**, without rewriting consumers. Reason: it delivers value immediately and reduces the "cold start" risk of a bad model (assumption about the data volume).

## Consequences
- + Functionality from day one; a safe, explainable baseline; smooth evolution; rules remain as a fallback on model degradation.
- − Rules are coarser than ML → lower accuracy at the start; dual maintenance (rules + model) during the transition period; risk of "rules forever" if a switching criterion is not set.
- Mitigation: explicit trigger metrics for the transition; shadow-mode (the model computes silently, we compare against the rules) before the actual switch.

## Considered / Rejected alternatives
- **ML right away on synthetic/third-party data** — rejected: domain-specific (exotic animals, a particular park), high risk of wrong predictions and loss of keepers' trust.
- **Defer AI functions until data is accumulated** — rejected: we lose early value, and the brief requires monetization immediately.

## Traceability
Brief goal (animal monitoring, profit growth in the absence of data) → **Evolvability/Accuracy** → "rules→AI" behind a facade (rules→AI strategy). Related ADRs: [ADR-010](ADR-010-human-in-the-loop-confidence-thresholds.md), [ADR-016](ADR-016-ai-marketing-and-scheduling-hitl-provider-abstraction.md), AIP.

**Mitigates risks:** R7 (see [risk-register](../03-views-and-perspectives/risk-register.md)).
