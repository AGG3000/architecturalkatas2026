# ADR-010: Human-in-the-loop and confidence thresholds for critical AI alerts (animal welfare)

## Status
Accepted

## Context
AI predicts animal health/anomalies (Q5) from telemetry and CV from cameras. Animal care is expensive, more so when an animal is sick; "healthy and happy animals" are needed (brief facts). A false alert wastes a keeper's resource; a **missed alert** may cost the life of a rare/venomous animal. GenAI/ML are nondeterministic. Affected: **Accuracy, Reliability**; the kata requirement is "validation & verification of AI results".

## Decision
Introduce **confidence thresholds** with three zones. **Default band** (calibrated confidence):

| Confidence | Action |
|---|---|
| **> 0.9** | **Auto action** — automatic action/escalation |
| **0.7 - 0.9** | **Human review** — an alert flagged "requires confirmation", a human decides |
| **< 0.7** | **Manual investigation** — abstain ("don't know") / fallback to rules, manual check |

**Thresholds are per-domain, not global:** for **safety/welfare/pricing** the bar is higher or a human is **mandatory regardless of score** — critical welfare decisions (euthanasia, isolation, change of feeding), the autonomous shuttle, publications, pricing policy go through a human even at confidence > 0.9. For low-risk ones (a content draft) auto is acceptable even at lower confidence. Reason: AI speeds up detection, but responsibility for the outcome remains with the human.

Thresholds are **calibrated** (confidence must reflect real accuracy) and **tuned with fitness functions** (too many false auto-actions → raise the threshold; overloaded confirmation queue → lower/prioritize).

## Consequences
- + Early disease detection without blind trust in the model; reduction of false alarms; auditability of decisions; support for validation/verification of AI results (V&V).
- − Operational load on keepers (confirmation queue); delay at the middle threshold; risk of "alert fatigue" with poor threshold calibration.
- Mitigation: threshold calibration by precision/recall on the golden-set; queue prioritization; when the model is uncertain — escalate conservatively (a false one is better than a miss for a critical case).

## Considered / Rejected alternatives
- **Fully autonomous AI actions without a human** — rejected: unacceptable for welfare, fails "validation of AI", legal/ethical risk.
- **Human only, no AI filtering** — rejected: does not scale to 200+ animals in 55 enclosures, early detection is lost.

## Traceability
Brief goal (animal health, control of the piranha population) → **Accuracy/Reliability + V&V** → confidence thresholds + human-in-the-loop. Related ADRs: [ADR-009](ADR-009-rules-to-ai-cold-start.md), [ADR-011](ADR-011-fixed-cameras-edge-cv-vs-cloud-cv.md), [ADR-016](ADR-016-ai-marketing-and-scheduling-hitl-provider-abstraction.md), AIP.

**Mitigates risks:** R6, R16 (see [risk-register](../03-views-and-perspectives/risk-register.md)).
