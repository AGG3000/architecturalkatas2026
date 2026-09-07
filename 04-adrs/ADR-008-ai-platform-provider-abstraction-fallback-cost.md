# ADR-008: AI Platform — provider abstraction + fallback + cost-monitor

## Status
Accepted

## Context
The focus of the kata is **how AI solves the Countess's problems** (brief fact, §G); the key kata requirements are dealing with AI uncertainty, validation/verification of results, and **alignment of the AI additions' characteristics with the architecture** (brief fact, §I). AI is applied end-to-end: CV counting of the piranha population and animal health, NL analytics, recommendations/quests, content generation (brief fact §D3 + assumptions about the applications). A model provider may **change, become more expensive, or shut down**, GenAI is nondeterministic, and the estate's budget is limited (brief fact: loss-making, §A) — the cost of AI must be controlled (assumption).

## Decision
Introduce an **end-to-end AI Platform (AIP) quantum = AI Gateway** — a single entry point for all AI calls (agents, quanta) to a **pool of providers** (e.g. GPT / Claude / Mistral for LLM; separate CV and autonomy vendors). Internal gateway components:
- **Prompt Adapter** — adapts the prompt/format to a specific provider (portability across GPT/Claude/Mistral).
- **Model Router** — **policy-based routing** across the pool: by task/complexity, cost (token arbitrage), capabilities (LLM/CV/reasoning), **health/latency** (health-based failover), not just "primary + fallback".
- **Prompt Registry** — versions of prompts/templates, model cards, rollback.
- **Capability contract** — a service/agent calls a **named capability** (e.g. `plan-visit`, `welfare-summary`), the gateway resolves it into a versioned **bundle {model+prompt+eval}**; the business code knows nothing about provider/model. (The term "capability" = a business capability as an addressable unit of the gateway; not to be confused with the "class" of a model in the Model Router.)
- **Executable invariant against lock-in** — the "no vendor lock-in" principle is not declared but **checked automatically**: an architectural fitness function in CI **blocks the import of a provider SDK into domain code** (all model/provider calls — only via AIP). A red gate instead of an "agreement".
- **Evaluation Engine** — offline golden-set in CI + online confidence thresholds, per-provider eval, fitness functions.

Plus cross-cutting gateway responsibilities: **provider abstraction + fallback**, **cost-monitor** (limits, inference cache, cheap default model + escalation), **rate-limiting/quotas per consumer**, **PII redaction on egress** to external models, **audit of every call**, unified **guardrails / protection against prompt injection**, management of provider keys (only in AIP). Keys and egress to the outside — only via AIP; the business quanta do not see providers.

Rationale: the AI domains are united by the unique characteristic "provider replaceability/control", hence a separate gateway quantum.

## Consequences
- Pros: replacing/adding a provider without changing domains, resilience to provider failure (fallback), cost control, a single point for guardrails and AI verification — closes AI uncertainty and provider change.
- Cons: an extra abstraction layer complicates integration and latency, risk of a sprawling number of providers, the fallback model may deliver different quality (output normalization needed).
- Mitigations: provider contract tests, fitness functions for inference quality, cache and confidence thresholds, human-in-the-loop for welfare-critical decisions; a "rules → AI" strategy for domains with no data at the start (assumption).

## Considered / Rejected alternatives
- **Hard binding to a single AI provider without abstraction** — rejected: the provider may become more expensive/shut down (AI uncertainty risk), vendor lock-in, a provider failure takes down all AI functions, no single point for cost/guardrails.
- **One provider + passive fallback (no router)** — rejected: we lose cost/quality arbitrage and health-based distribution; switching only on a full failure.
- **Direct calls to providers from domains/agents** — rejected: duplication of adapters/keys, no single point for guardrails/PII-redaction/audit, insecure.
- **Own ML stack from scratch (train-and-serve ourselves)** — rejected: unrealistic given the estate's budget/timelines, no data at the start, wrong level of detail for the kata.

> Caveat: policy-based routing applies to LLM/CV tasks; for **safety-critical** ones (autonomous shuttle) the provider is fixed (a certified vendor), without on-the-fly arbitrage.

## Phasing (MVP → roadmap)
- **MVP (from the very start):** **Prompt Adapter** + **Evaluation Engine** (eval-harness / per-provider / fitness) + provider abstraction/fallback + basic guardrails, cost-monitor, PII redaction, audit, key management + **lightweight prompt versioning** (in the repository/config). Reason: the Prompt Adapter and Evaluation Engine **directly reduce vendor lock-in and model quality degradation** — a baseline risk under multi-provider, so they are introduced immediately.
- **Roadmap (as load and the number of AI scenarios grow):** a full **AI Governance / experiment platform** — **A/B testing** of models/prompts, a **full Prompt Registry**, **Experiment Tracking**, an advanced data-driven Model Router. Reason: redundant and expensive while there are few scenarios; introduced when there is something to A/B-test and something to manage at scale.

*(Clarification: at MVP — lightweight prompt versioning; a full Prompt Registry with experiment tracking is a roadmap phase.)*

## Traceability
Brief goal "AI solves the Countess's problems" + requirements for AI uncertainty/validation, alignment of characteristics, and cost control → characteristics Portability·Observability·Cost-control·Accuracy → AIP with provider abstraction, fallback, and cost-monitor. Related ADRs: [ADR-001](ADR-001-event-driven-architecture-and-quanta.md), [ADR-005](ADR-005-cqrs-analytics-operations-split.md).

**Mitigates risks:** R5 (see [risk-register](../03-views-and-perspectives/risk-register.md)).
