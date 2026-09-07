# ADR-018: AI Governance Framework (risk classification + mandatory controls)

## Status
Accepted

## Context
The solution is AI-heavy (Q5, Q6, Q9, Q10, Q11, Q12 + Fixed Cameras). Technical controls already exist but are scattered: guardrails and versions (AIP, [ADR-008](ADR-008-ai-platform-provider-abstraction-fallback-cost.md)), human-in-the-loop (see [ADR-010](ADR-010-human-in-the-loop-confidence-thresholds.md), [ADR-016](ADR-016-ai-marketing-and-scheduling-hitl-provider-abstraction.md)), privacy (see [ADR-013](ADR-013-privacy-surveillance-and-mobile-telemetry.md)), V&V (fitness functions). What is missing is a **unified AI governance framework**: classification by risk level, a model lifecycle policy, transparency, bias/fairness, accountability, and AI-incident response. The context sharpens the risk: **child visitors and venomous animals**, video surveillance, autonomous transport → some AI falls into the **high-risk** category in the spirit of the EU AI Act class (assumption on applicability — the jurisdiction is not stated in the brief).

## Decision
Introduce an **AI Governance framework** on top of the AIP: every AI application receives a **risk level** and a mandatory set of controls by level.

| Level | Examples | Mandatory controls |
|---|---|---|
| **High-risk** | autonomous shuttle (Q10), animal welfare decisions (Q5), video surveillance | human-in-the-loop/oversight, ODD + fail-safe, decision audit trail, model card, deployment approval gate, enhanced monitoring, incident plan |
| **Medium** | recommendations/quests (Q6), schedules (Q12), sentiment (Q9) | guardrails, bias/fairness check, fitness functions, human approval for decisions affecting people |
| **Low** | content drafts (Q11), NL analytics (Q3) | guardrails/brand-safety, transparency (AI labeling), human approval of publications |

Common mandatory elements: **model registry + versions + model cards**, an **approval gate** for model rollout (eval passed thresholds), **rollback/deprecation**, an **audit trail** of AI decisions, **transparency** (labeling AI content), **data governance** (consent for using data for training, lineage), **accountability** (an owner for each AI domain), and **AI-incident response**. Implemented as policies in the AIP (control plane), not as a new quantum.

## Consequences
- Pros: a coherent, verifiable framework; covers uncertainty, V&V, and reconciliation of AI characteristics; reduced regulatory/reputational risk; audit-readiness.
- Cons: process overhead (approval gate, model cards, audit) slows rollout; bias/fairness requires labeled data and metrics; the incident process must be maintained.
- Mitigations: the strictness of controls is **proportional to the risk level** (low — light, high — strict); automation of the gate/audit in CI and the AIP; reuse of the already-existing HITL/fitness/registry.

## Considered / Rejected alternatives
- **Leave controls scattered (no unified framework)** — rejected: gaps (bias, transparency, risk classification, incident), nothing to prove AI manageability at audit.
- **A single maximum control for all AI** — rejected: excessive for low-risk (content drafts), slows things needlessly; a risk-based approach is more effective.

## Traceability
Theme §G (AI-assisted) + uncertainty/V&V requirements and characteristic reconciliation + context §B (venomous animals, children) → **Governance/Safety/Compliance** → risk classification + mandatory controls by level. Details — `../05-ai/governance.md`. Related ADRs: [ADR-008](ADR-008-ai-platform-provider-abstraction-fallback-cost.md), [ADR-010](ADR-010-human-in-the-loop-confidence-thresholds.md), [ADR-013](ADR-013-privacy-surveillance-and-mobile-telemetry.md), [ADR-015](ADR-015-autonomous-shuttle-and-data-mule.md), [ADR-016](ADR-016-ai-marketing-and-scheduling-hitl-provider-abstraction.md).

**Mitigates risks:** R6, R7, R13 (see [risk-register](../03-views-and-perspectives/risk-register.md)).
