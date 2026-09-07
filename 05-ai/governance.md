# AI Governance perspective

A coherent AI governance frame on top of the AIP: how we keep AI under control — by risk level. The decision — [ADR-018](../04-adrs/ADR-018-ai-governance-framework.md); mechanisms — [ADR-008](../04-adrs/ADR-008-ai-platform-provider-abstraction-fallback-cost.md) (AIP), [ADR-010](../04-adrs/ADR-010-human-in-the-loop-confidence-thresholds.md) (HITL), [ADR-013](../04-adrs/ADR-013-privacy-surveillance-and-mobile-telemetry.md) (privacy); verification — [validation-verification.md](validation-verification.md).

## Risk classification of AI applications (EU AI Act class — assumption of applicability)

| Level | Applications | Why |
|---|---|---|
| **High-risk** | autonomous shuttle (Q10); animal welfare decisions (Q5); video surveillance (cameras); **predictive maintenance of rides (Q13)** | physical safety of people/children; irreversibility of decisions; biometrics/surveillance; **safety of historic rides → AI flags, a human clears operation** |
| **Medium** | route-quest/recommendations (Q6); staff/feeding scheduling (Q12); sentiment (Q9); **dynamic pricing (Q1/Q6)** | affect people (visitors/staff/animals), but not safety-critical; prices → fairness/transparency + human-approved policy |
| **Low** | content drafts (Q11); NL analytics (Q3); visitor support assistant (RAG) | internal/draft, under human approval |

*(RAG assistant: staff version — **medium** (care/safety → human-in-the-loop + grounding with citations); visitor version — **low** (grounding is mandatory against hallucinations). See [ADR-019](../04-adrs/ADR-019-rag-knowledge-assistant.md).)*

*(Agents by stakeholder ([ADR-023](../04-adrs/ADR-023-agentic-layer.md), [ADR-024](../04-adrs/ADR-024-lakehouse-as-agent-memory.md)): Visitor/Operations/Management — **medium** (effectful actions only through human-approval + typed least-privilege tools + audit); the **Animal Agent** for welfare interventions — **high** (veterinarian confirmation is mandatory); the autonomous shuttle — **high**. Writes to shared memory — under consent/PII minimization.)*

The strictness of controls is **proportional to the level**.

## Mandatory controls (by governance dimension)

- **Human oversight / accountability.** Every AI domain has an owner; high/medium decisions pass through a human (keeper Q5, manager Q12, curator Q11, teleoperator Q10). Responsibility for the outcome lies with the human, not the model.
- **Model lifecycle.** Model registry + versions + **model cards** (purpose, data, metrics, limitations); **rollout gate** (eval passed the fitness-function thresholds); **rollback/deprecation**; fallback provider (AIP).
- **Guardrails & policy enforcement.** A single point in the AIP + domain-specific: brand-safety (Q11), **quest reward economic guardrails** (Q6 — rewards only from a human-approved pool and within the budget/discount limits of Q1), solver hard-constraints (Q12).
- **Bias / fairness.** Checking for skew in models that affect people: recommendations/quests (do not disadvantage groups of visitors), schedules (shift fairness), content. Fairness metrics + periodic audit.
- **Transparency / disclosure.** Labeling of **AI-generated content** (Q11); decision explainability (explainable "rules→AI" baseline; NL analytics shows the source/query).
- **Data governance.** Consent for using data for training; PII minimization (pseudonymous ticket_id); lineage of inference/training data; edge video privacy.
- **Audit trail.** Logs of AI decisions and their human confirmations (welfare events 2–3 years), eval logs, audit of access to the identity linkage (Q8) — for investigations and regulatory reporting.
- **Monitoring & AI-incident response.** Continuous quality/drift monitoring (V&V); on an incident (quality drop, harmful output, spoofing) — the process: detect → roll back to fallback/rules → notify the owner → post-mortem. For agents, quality is tracked via **Task Success Rate · Tool Error Rate · Human Override Rate** (a rise in human cancellations = an early degradation signal); on exceeding thresholds — auto-mitigation (model/prompt rollback, prompt/tool revision, strengthening human-approval, narrowing permitted actions). See [validation-verification](validation-verification.md).

## "Risk level → controls" matrix

| Control | Low | Medium | High |
|---|---|---|---|
| Guardrails | ✅ | ✅ | ✅ |
| Human approval | publications | affecting people | always + oversight |
| Model card + rollout gate | basic | ✅ | ✅ strict |
| Bias/fairness audit | — | ✅ | ✅ |
| Transparency (AI labeling) | ✅ | ✅ | ✅ + explainability |
| Decision audit trail | basic | ✅ | ✅ (long retention) |
| Incident plan | general | ✅ | ✅ + fail-safe |

## What this covers
A single frame covers: risk classification, bias/fairness, transparency/disclosure, model-lifecycle policy, data-governance/consent-for-training, and AI-incident response — AI manageability under safety and regulatory risks (**AI Governance**).
