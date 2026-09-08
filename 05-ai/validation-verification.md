# AI validation and verification

A key requirement of the kata. GenAI is non-deterministic → an explicit trust loop is needed.

## How we verify that AI works
- **Offline eval-harness** + golden-set on model/prompt releases.
- **Fitness functions** in CI/CD: inference accuracy, latency, cost per request.
- **Shadow-mode**: AI "drives" silently, a human cross-checks — before going to prod (especially Q10 autonomy).

## Human-in-the-loop (for critical AI)
- **Q5** — welfare alerts: confidence thresholds + keeper confirmation.
- **Q11** — publications: approval queue (a human publishes).
- **Q12** — schedules: the solver holds hard-constraints (labor/veterinary norms), a human approves.
- **Q10** — autonomy: teleoperation + safety-driver on MVP.

## How we detect a failure in prod
- Continuous monitoring of inference quality + drift; alerting when metrics exceed thresholds; incident logging.
- The deterministic is validated simply; the non-deterministic — via a separate loop (eval + guardrails + human gate).

## Concrete fitness functions by AI application

Thresholds are examples for calibration on real data (numbers = assumption), but the metric structure is fixed. A red threshold = alert/rollback/switch to fallback.

*Metrics in business terms:* **recall** = share of real cases caught (a miss is costly), **precision** = share of correct triggers (fewer false alarms), **F1** = their balance, **MAE/MAPE** = average forecast/count error, **PSI** = how far the data has "drifted" from the training data (drift). Definitions — in the [glossary](../glossary.md).

| Quantum / application | Fitness function (metric) | Threshold example | Response on violation |
|---|---|---|---|
| **Q5 — health/population CV** | Inference accuracy on golden-set (F1 of anomaly detection; piranha count — **robust estimate: median over multi-view + confidence interval**, excluding feeding time) | F1 ≥ 0.90; count error ≤ 10%; CI width ≤ threshold | block model release; rollback; a sharp count jump → welfare-review, not an auto-conclusion |
| **Q5 — drift** | Data/prediction drift (PSI over inputs, shift of the confidence distribution) | PSI < 0.2; share of low-confidence < 15% | alert → retraining; raise the share of human-review |
| **Cross-cutting — confidence calibration** | Calibration error: **ECE** (Expected Calibration Error) + **Brier score** + reliability curve on the golden-set — checks that the stated confidence reflects real accuracy (in the 0.9 bin the share of correct ≈ 90%). Measured for **any** confidence-scored output (CV Q5, LLM answer, forecast Q20/Q21) | **ECE ≤ 0.05**; Brier ↓ vs baseline; max bin deviation from the diagonal ≤ 0.1 | **block model/prompt release** (calibration-gate); re-calibrate (temperature/Platt scaling); until fixed — shift routing per [ADR-010](../04-adrs/ADR-010-human-in-the-loop-confidence-thresholds.md) more conservatively (more into human-review) |
| **Agent — memory (write-back)** | Feedback-loop self-reinforcement: drift of the memory-derivative distribution (**PSI on self-generated writes**) + share of self-generated vs human-verified labels + rejected write-guard entries | derivative PSI < 0.2; share of self-verified does not grow unchecked; accepted anomalous writes = **0** | self-reinforcement alert → ↑ human-review; **roll back the poisoned memory batch by provenance** ([ADR-024](../04-adrs/ADR-024-lakehouse-as-agent-memory.md)) |
| **Q5 — welfare alerts** | Recall of critical conditions (a missed illness costs more than an FP) + share confirmed by the keeper | recall ≥ 0.95; precision ≥ 0.70 | lower the confidence threshold, strengthen the human gate |
| **Q3 — heatmap/analytics** | Latency of heatmap/NL analytics query (p95) + data freshness | p95 < 2 s; data lag < 5 min | scale the read-side; degrade to cache |
| **Q3 — queue CV** | Accuracy of queue-length/wait-time estimate vs reference | error ≤ 15% | recalibrate camera/zone model |
| **Q6 — quest/recommendations** | Quest conversion (share of starters → finishers) + return uplift (A/B vs control) | completion ≥ 30%; uplift > 0 statistically significant | roll back the recommendation strategy |
| **Q6 — quest reward economic guardrails** (AI selects a reward only from a human-approved pool and within the budget/discount limits of Q1, rather than assigning arbitrarily) | Share of rewards outside the pool / beyond economic limits | 0 economics violations | block reward auto-generation → manual approval |
| **Q10 — autonomy (shadow-mode)** | Disengagement rate (operator interventions per 1000 km/h) + share of fail-safe stops | trend ↓; disengagement below target before driverless | do not take out of supervised; keep the safety-driver |
| **Q10 — perception** | Perception accuracy (obstacle detection, FN per obstacle) within the ODD | obstacle FN → 0 within the ODD | halt the rollout; expand only by statistics |
| **Q11 — content** | Approval-rate (share of drafts accepted by a human without edits) | ≥ 60% (otherwise brand-voice tuning) | tune prompts/templates |
| **Q11 — brand-safety** | Share of content rejected by moderation/guardrails (inappropriate, visitor faces) | 0 published violations | strengthen guardrails; tighten the filter |
| **Q12 — schedules** | Number of hard-constraint violations (labor/veterinary norms) + quality (overtime/idle) | 0 hard-constraint violations | reject the plan; solver restart |
| **Q13 — predictive maintenance** | Recall of mechanism-failure forecast + **lead-time** of warning (how far before failure) + missed safety incidents | recall of critical failures ≥ 0.95; lead-time ≥ target; missed safety incidents = **0** | more conservative thresholds → earlier inspection; auto-clearance is blocked (only a human clears) |
| **AIP — cost** | Cost per request + inference cache hit rate | within budget; hit-rate ≥ target | model escalation/cost-monitor alert |
| **Agent — task (prod-eval)** | Task-success (LLM-judge on a sample + human labeling) + tool-error rate + **human-override rate** (leading indicator) | success ≥ target; override/tool-error ↓ | agent rollback (prompt/version); tighten whitelist/HITL |
| **Agent — cost/chain** | Cost-per-task + number of steps per session | within budget; steps ≤ limit | step limit/degradation; cost-monitor alert |
| **PWA — accessibility** | WCAG 2.2 AA violations (axe-core / Lighthouse-a11y) on key flows | 0 critical; score ≥ target | block the frontend release; fix contrast/aria |

**Key principle for sensitive domains:** for Q10 (autonomy), Q12 (hard-constraints), and **Q13 (ride clearance for operation)** the allowable number of safety/welfare invariant violations = **0** — this is not an optimizable metric but a gate; for Q13 AI only flags for inspection, and a human clears operation.

## Confidence calibration — how we verify "0.9" = 90% (gate)

All decision routing runs on confidence thresholds ([ADR-010](../04-adrs/ADR-010-human-in-the-loop-confidence-thresholds.md)), so an uncalibrated score voids the three-zone logic. We **operationalize** calibration rather than declaring it:

- **Metric:** **ECE** (Expected Calibration Error) — the weighted-average gap between "stated confidence ↔ actual share correct" across bins; complemented by **Brier score** and a **reliability curve** (the diagonal = perfect calibration). ECE answers exactly the question "in the 0.9 bin, are ~90% actually correct?".
- **Threshold & gate:** **ECE ≤ 0.05**, max bin deviation ≤ 0.1 — as a **calibration-gate in the CI eval-harness**: a model/prompt release failing the threshold is **blocked** (alongside the accuracy/latency fitness functions). On failure — re-calibrate (temperature/Platt scaling); until calibrated, [ADR-010](../04-adrs/ADR-010-human-in-the-loop-confidence-thresholds.md) thresholds temporarily shift toward human-review.
- **Where in the pipeline:** (1) **offline** — on the golden-set before rollout (blocking gate); (2) **in prod** — continuous calibration drift (reliability-curve shift), "confident-but-wrong" → miscalibration signal → retraining/re-calibration.
- **Ground-truth (what we measure against):** the same **golden-set** labels (curation below) — verified outcomes: vet confirmation for welfare (Q5), manual/reference count for queues (Q3) and piranha (Q5; median over multi-view + CI as the reference, excluding feeding time), actual failure/inspection for Q13. Calibration is only as honest as the golden-set ground-truth.

## Agent quality monitoring (in prod)
The quality of AI services is tracked continuously via three metrics understandable to both engineer and business:

- **AI Task Success Rate** — how often AI successfully completes its task. Assessed automatically (**LLM-as-a-Judge** on a test sample) and by selective human review. The goal is to keep success no lower than an agreed threshold.
- **Tool Error Rate** — share of requests where AI failed to correctly use tools or external systems. A rise may indicate integration problems, incorrect prompts, or changes in external APIs.
- **Human Override Rate** — how often staff reject or correct AI recommendations. This is the **earliest indicator of degradation**: user trust drops before it becomes visible in other metrics.

**Corrective Actions.** If quality falls below the target level, error counts rise, or users increasingly cancel AI recommendations — the platform performs corrective actions: rollback to a previous model or prompt version; revision of the agent's instructions and tools; strengthening Human-in-the-Loop control; restricting the agent's set of permitted actions until quality recovers.

## Golden-set governance
The eval gate is only as honest as its golden set:
- **Owner** — each capability's golden set has a named owner (vet for Q5 welfare, ops for Q3, curator for Q11).
- **Versioning** — golden sets are versioned; a model is evaluated against a pinned version and the result is stored with the model+prompt version (reproducible).
- **Rare/critical-class coverage** — deliberately over-sample rare-but-critical cases (illness, cold-blooded escape, a bite); recall ≥ 0.95 is meaningless if the set has 2 positives per 1000 → a **coverage target per critical class + stratified metrics**, not just aggregate F1.
- **Leakage prevention** — train/golden split **by time and by entity** (no same-animal / same-session leakage between train and eval).
- **Growth & re-validation** — keeper/vet confirmations become new labels (labels are a side-effect of the job); the golden set is re-validated each season and on drift (PSI), with relabeling on distribution shift.
