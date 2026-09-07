# ADR-016: AI Social Marketing (Q11) and AI Scheduling (Q12) — human-in-the-loop + provider abstraction

## Status
Accepted *(decision accepted; Q11/Q12 rollout — roadmap R1)*

## Context
Two back-office AI domains with a shared pattern for managing nondeterminism. **Q11 (Social Marketing):** AI mines highlight moments from cameras (piranha feeding, venomous-animal hunts), generates posts (LLM), and schedules multi-platform posting — hitting the brief's goals of "growing visitor numbers and return rate". The estate's brand is fragile and the subject matter sensitive (venomous animals, children). **Q12 (Scheduling):** AI optimization of staff schedules (rostering by demand forecast), feeding schedules (welfare × crowd × content), and maintenance windows — the goals "where to deploy staff", "healthy animals". An error in a welfare/labor schedule is unacceptable. Social APIs and models change/become obsolete. Affected: **Evolvability, AI-provider-agnostic, Optimizability, low-SLA (reputation/welfare-critical)**; the kata's requirement — "validation of AI".

## Decision
A shared loop for both. **Q11:** AI **prepares** content → **approval queue** → a human publishes; all social platforms and the content model sit behind a **provider abstraction** (AIP); highlight-mining reuses the cameras' edge-CV (only animal/territory content, no faces). **Q12:** an LLM formalizes goals/explanations, a **deterministic solver/OR holds the hard constraints** (labor rules, veterinary requirements, capacity), a human **approves** the draft; schedules are batch, non-real-time, behind an AI facade. Rationale: nondeterministic GenAI cannot be let loose on reputation and welfare without a human and without provider replaceability.

## Consequences
- + Scale of content and planning with a small team; on-brand and legally safe; welfare-awareness; resilience to changes of social API/model/solver; explainability of decisions; a direct answer to V&V.
- − Human-in-the-loop limits speed/volume (the approval queue is an operational load); the provider abstraction complicates integration; a risk of an "AI-generic" tone; Q12's dependence on the quality of the Q3 forecast.
- Mitigation: brand-voice templates; auto-publishing only of low-risk formats ("feeding at 15:00") with post-moderation; hard constraints as solver constraints, not a "request" to the LLM; fitness functions (engagement→visits; overtime/idle time, welfare incidents).

## Considered / Rejected alternatives
- **Fully autonomous posting without a human (Q11)** — rejected: faster, but reputationally dangerous, fails "validation of AI".
- **Direct integration with a single social network without an abstraction (Q11)** — rejected: vendor lock-in, fragility to API changes.
- **Generating schedules with a single LLM without a solver/constraints (Q12)** — rejected: nondeterministic and dangerous for welfare/labor rules.

## Traceability
Brief goals (visitor growth/return, staff deployment, healthy animals) → **Evolvability/AI-provider-agnostic/Optimizability + V&V** → human-in-the-loop + provider abstraction + a solver with hard constraints (Provider pattern, fitness functions). Related ADRs: [ADR-009](ADR-009-rules-to-ai-cold-start.md), [ADR-010](ADR-010-human-in-the-loop-confidence-thresholds.md), [ADR-011](ADR-011-fixed-cameras-edge-cv-vs-cloud-cv.md), AIP.

**Mitigates risks:** R20 (brand-safety of Q11 AI content) (see [risk-register](../03-views-and-perspectives/risk-register.md)).
