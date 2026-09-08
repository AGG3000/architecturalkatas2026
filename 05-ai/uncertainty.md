# Handling AI uncertainty

A key requirement of the kata. We distinguish **two facets**:
- **A. Uncertainty in the AI outputs themselves** (model / data / future) — how the system behaves when the model is unsure, the data is bad, or the forecast is inherently unreliable.
- **B. Provider/platform uncertainty** — the model/provider may change, get more expensive, shut down, or update.

## A. Uncertainty in AI outputs (model / data / future)

| Type | What it means | Our handling | Mechanism |
|---|---|---|---|
| **Model uncertainty** | AI is **unsure** of the answer | confidence thresholds (3 zones: auto / human-in-the-loop / ignore), **abstain "I don't know"** (RAG grounding), fallback to rules; confidence calibration | [ADR-010](../04-adrs/ADR-010-human-in-the-loop-confidence-thresholds.md), [ADR-019](../04-adrs/ADR-019-rag-knowledge-assistant.md), [ADR-009](../04-adrs/ADR-009-rules-to-ai-cold-start.md) |
| **Data uncertainty** | the model is **confident, but the input is bad** (broken sensor/noise) | input validation/quality, **sensor health/outlier detection**, **cross-confirmation** (thermal + MQTT + RGB), don't act on a single reading, drift monitoring (PSI) | Edge AI Platform, [ADR-011](../04-adrs/ADR-011-fixed-cameras-edge-cv-vs-cloud-cv.md), [validation-verification.md](validation-verification.md) |
| **Future uncertainty** | the forecast is **inherently unreliable** | forecast **intervals/confidence bands**, short horizons, continuous re-forecast, smoothing, human review for impactful decisions, protection against self-fulfillment | [ADR-020](../04-adrs/ADR-020-crowd-prediction-ai.md), [ADR-021](../04-adrs/ADR-021-dynamic-pricing-ai.md) |

**Principle:** the higher the uncertainty and the cost of error, the stricter the gate — from "act automatically" (low, reversible) to "only after human confirmation" (welfare/safety/prices). Abstaining ("I don't know") is better than a confident error.

### Mechanism: Confidence Score + Explainability
Every AI output is a **triplet `{result, confidence score, explanation}`**, not a "bare" value. This is exactly the operationalization of facet A:

**Confidence Score (how confident → what to do):**
- a calibrated confidence estimate per output (CV detection, LLM answer, forecast);
- **routing by threshold** ([ADR-010](../04-adrs/ADR-010-human-in-the-loop-confidence-thresholds.md)) — default band: **>0.9 → auto action** · **0.7–0.9 → human review** · **<0.7 → manual investigation** (abstain/rules); thresholds are **per-domain** (safety/welfare/prices — a human is mandatory regardless of score);
- for **future** uncertainty the score is not a point but an **interval/confidence band**; for **data** uncertainty the score incorporates the **input quality signal** (broken sensor/noise → low confidence regardless of the model);
- **calibration is monitored and gated** — an **ECE** metric (+ Brier / reliability curve) with a threshold as a **calibration-gate in CI**: confident-but-wrong = miscalibration → block the release + re-calibrate (temperature/Platt), see [validation-verification.md](validation-verification.md#confidence-calibration--how-we-verify-09--90-gate).

**Explainability (why this answer → what to check it with):**
- every decision comes with a **reason/evidence**: CV — what and where was detected (bbox / thermal zone); RAG/LLM — **source citations** ([ADR-019](../04-adrs/ADR-019-rag-knowledge-assistant.md)); forecast — factor contributions; "rules→AI" — the rule is itself explainable;
- the explanation makes **human-in-the-loop a real validation**, not a "blind approve";
- it feeds **transparency/audit** governance ([ADR-018](../04-adrs/ADR-018-ai-governance-framework.md)): AI-content labeling, explainable decisions, audit trail.

**Together:** *confidence* decides "act / ask / abstain," *explainability* gives the human "what to check it with." Both are **logged** (audit) and serve as a **fitness signal** (share of low-confidence, share rejected by humans, calibration).

## B. Provider/platform uncertainty

## Summary: uncertainty type → mitigation
| Uncertainty | Mitigation | Mechanism |
|---|---|---|
| Provider changes / a better one appears | provider abstraction, unified interface | AIP (ADR-008) |
| Provider raises prices | cost-monitor, cheap default model + escalation, cache | AI Gateway |
| Provider shuts down | fallback provider | AIP |
| **Model updated → behavior drift** | pin version + **eval-gate before switching** | Evaluation Engine |
| **Model deprecated (deprecation)** | migration plan on notice, contract tests | AIP + registry |
| No data at the start | "rules → AI" strategy | ADR-009 |

## Details

**Provider switch / better model.** All AI calls sit behind the **provider abstraction (AIP)**: a unified interface, model/provider configurable. The provider pattern applies to social APIs (Q11), CV models (Q5/cameras), and generation (Q6/Q11/Q12).
> **Important: switching is not free.** It is not "flip a config": GPT/Claude/Mistral react differently → a **Prompt Adapter** is needed + a **repeat per-provider eval** on the golden-set before rollout. The abstraction removes code coupling, but does not waive validation.

**Price growth.** Cost-monitor at the AIP gateway; cheap default model + escalation to the expensive one only when necessary; inference cache.

**Provider shutdown.** A fallback provider behind the same abstraction. For the autonomous shuttle (Q10) the **stronger lock-in** is honestly noted (physical platform) → choice of a certified vendor + integration migration plan.

**Behavior drift on model update.** Uncertainty is not only commercial: a new model version may **regress/change outputs**. Mitigation: **version pinning** + **eval-gate** (a new version/provider is rolled out only if it passes the fitness-function thresholds on the golden-set) + drift monitoring in prod (see [validation-verification.md](validation-verification.md)).

**Model deprecation/sunset.** More common than a full shutdown: a model is taken out of support. Mitigation: provider contract tests, responding to a deprecation notice per the migration plan (the registry stores versions/model cards).

**No data at the start.** The **"rules → AI"** strategy: we begin with deterministic rules and switch to a model as data accumulates (ADR-009).

## Switching triggers (switching policy)
We change the model/provider **on an objective signal**, not manually:
- quality below threshold (fitness function / rising share of low-confidence),
- cost per request above the limit (cost-monitor),
- deprecation notice or provider failure/degradation,
- significant drift (PSI over threshold).
Any switch passes the **eval-gate** and, for critical cases, human confirmation.

## Diagram
The AI Gateway with provider-substitution points (Prompt Adapter · Model Router · Evaluation Engine · Prompt Registry) — [../diagrams/ai-gateway.md](../diagrams/ai-gateway.md).
