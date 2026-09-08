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

**Thresholds are derived from the cost of error, not from a "0.9" convention.** For each domain we weigh the cost of a **false positive (FP)** against a **false negative (FN)**: for welfare a miss (a dead rare/venomous animal) is many times costlier than a false alarm → the asymmetry is **FN ≫ FP** → the bar is shifted toward **escalation/human review** (maximizing recall, conservatively). For low-risk ones (a content draft) an FP is cheap → a higher auto bar is acceptable. So the specific threshold numbers are a **consequence of the domain's cost-of-error matrix**, and are revised together with it (e.g. welfare recall in [validation-verification](../05-ai/validation-verification.md)).

**Cost-of-error matrix** (illustrative — calibrate on real data):

| Domain | Cost of a miss (FN) | Cost of a false alarm (FP) | ~FN:FP | Threshold bias |
|---|---|---|---|---|
| Ride safety (Q13) | injury (catastrophic) | inspection time | ~100:1 | 0 missed = hard gate; human clears |
| Animal welfare (Q5) | death of a rare/venomous animal | keeper time | ~20:1 | maximize recall; human mandatory |
| Content brand-safety (Q11) | brand/reputational hit | wasted draft | ~5:1 | approval queue before publish |
| Dynamic pricing (Q1) | lost revenue / perceived unfairness | minor | ~3:1 | corridors + human-approved policy |
| Concierge suggestion (Q6) | a weak suggestion | a weak suggestion | ~1:1 | auto is fine |

Rule: the higher the FN:FP ratio, the lower the escalation bar (more goes to a human). The numbers are assumptions — the point is that thresholds are **derived** from this matrix, not picked by convention.

Thresholds are **calibrated** (confidence must reflect real accuracy) and **tuned with fitness functions** (too many false auto-actions → raise the threshold; overloaded confirmation queue → lower/prioritize).

**Calibration is a verifiable gate, not an assumption.** Routing by these thresholds is only meaningful with a calibrated score, so calibration is measured with **ECE (Expected Calibration Error)** + Brier + a reliability curve on the golden-set and acts as a **blocking calibration-gate in CI** (ECE ≤ 0.05): a release whose confidence does not reflect real accuracy is not rolled out; until re-calibration (temperature/Platt) the thresholds temporarily shift toward human-review. Details and ground-truth — [validation-verification.md](../05-ai/validation-verification.md#confidence-calibration--how-we-verify-09--90-gate).

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
