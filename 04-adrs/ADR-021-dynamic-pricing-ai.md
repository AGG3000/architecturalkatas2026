# ADR-021: Dynamic Pricing AI — dynamic pricing of tickets and passes

## Status
Accepted

## Context
Goals §D/§F: profitability, attendance growth, and smoothing peaks. Static prices do not optimize revenue and do not distribute the crowd. **Dynamic pricing** of tickets/passes by demand/load/time/segment raises revenue and "spreads out" attendance (cheaper off-peak). But a price that affects people is **ethically and reputationally sensitive**: children, family passes, the risk of a perception of unfairness and discrimination. Pricing is not specified in the brief → **(assumption)**.

## Decision
Introduce **Dynamic Pricing as an AI capability** at the junction of **Q1** (tickets) and **Q6** (growth), fed by the demand forecast ([ADR-020](ADR-020-crowd-prediction-ai.md) / Q3). The AI proposes a price **within a human-approved policy**: min/max corridors, protection of family passes, **pricing by time/load, not by identity** (no personal discrimination). **Transparency:** we show the visitor the reason ("off-peak-time discount"). Governance — **medium-risk**: a fairness audit + human approval of the pricing policy (see [ADR-018](ADR-018-ai-governance-framework.md)).

## Consequences
- Pros: revenue growth, smoothing peaks (off-peak discounts → better crowd distribution and UX), flexible passes; reuses the demand forecast.
- Cons: reputational/ethical risk (may be perceived as unfairness); complexity; regulatory constraints on pricing; a risk of reducing accessibility.
- Mitigations: price corridors (min/max), protection of the family-pass as a social minimum, pricing by time/load (not by profile), transparency of the reason, a human-approved policy, a fairness audit (governance), fitness — revenue/load vs complaints.

## Considered / Rejected alternatives
- **A fully autonomous AI pricing without corridors/a human** — rejected: reputationally and ethically unacceptable, regulatory risk.
- **Personalized prices by visitor profile** — rejected: direct discrimination; we tie the price to time/load, not to identity.
- **Static prices** — rejected: they don't hit profitability and don't smooth peaks (the brief's goal).

## Traceability
§D/§F (profitability, growth, load distribution) → Revenue-optimization + Fairness/Transparency → dynamic pricing within corridors under a human policy. Related ADRs: [ADR-006](ADR-006-pci-ticketing-isolation-external-psp.md) (Q1/PSP), [ADR-018](ADR-018-ai-governance-framework.md) (governance/fairness), [ADR-020](ADR-020-crowd-prediction-ai.md) (demand forecast).

**Mitigates risks:** R13 (see [risk-register](../03-views-and-perspectives/risk-register.md)).

**V&V:** revenue/load vs complaints + a periodic fairness audit — metrics/thresholds in [validation & verification](../05-ai/validation-verification.md).
