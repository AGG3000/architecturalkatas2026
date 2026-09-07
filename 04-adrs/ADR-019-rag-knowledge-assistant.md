# ADR-019: RAG Assistant (knowledge base) for staff and visitor support

## Status
Accepted

## Context
Staff need fast access to heterogeneous **internal documents**: animal and plant care protocols, keeper/gardener SOPs, maintenance instructions for the 18th-century rides. Visitors need **support** (tickets, hours, navigation, animal facts). Hardcoded FAQ/rules do not scale to heterogeneous documents; a "bare" LLM hallucinates without grounding to sources. Theme §G (AI-assisted); goals §D (healthy animals — quality care; growth/return — good support/UX). *(assumption: the existence of a document corpus is not directly stated in the brief.)*

## Decision
Introduce a **RAG assistant as a capability behind the AIP** (not a new quantum): internal documents are indexed into a **vector knowledge base**; answers are generated via retrieval + **mandatory source citation** (grounding, "I don't know if it's not in the base"). Two interfaces in the mobile app: (1) a **staff assistant** (care/SOP/manuals); (2) a **visitor support assistant** (tickets/navigation/facts). Embeddings and the LLM go through the AIP provider abstraction. **Visit recommendations remain ML (Q6)**; RAG only **enriches the content of cards** ("tell me about this animal") but does not serve as the recommendation engine.

## Consequences
- Pros: fast access to knowledge, less load on staff, better support/UX → visitor return; reuses already-collected documents and care data; grounding + citations reduce hallucinations.
- Cons: the index must be kept current; a risk of stale/contradictory sources; the cost of embeddings/inference; hallucinations on poor retrieval.
- Mitigations: source citation + refusal when not in the base; **human-in-the-loop for staff-critical answers** (veterinary/safety); versioning and regular index refresh; guardrails; fitness functions (grounding-rate, share of answers with a source).

## Considered / Rejected alternatives
- **Fine-tune an LLM on internal data** — rejected: expensive, becomes stale fast, hard to update, a risk of "baking" data into the weights, worse source attribution.
- **Hardcoded FAQ / rules** — rejected: does not scale to heterogeneous documents, expensive to maintain, does not cover staff protocols.
- **RAG as the recommendation engine** — rejected: recommendations are ML on behavior (Q6); RAG is inappropriate for this (a stretch), permissible only as content enrichment.

## Traceability
§G (AI-assisted) + §D (animal/plant care = health; support = growth/return) → **Evolvability/Accuracy/Usability** → a RAG assistant behind the AIP with grounding. Governance: staff — medium, visitor — low (see `../05-ai/governance.md`). Related ADRs: [ADR-008](ADR-008-ai-platform-provider-abstraction-fallback-cost.md), [ADR-010](ADR-010-human-in-the-loop-confidence-thresholds.md), [ADR-018](ADR-018-ai-governance-framework.md).

**Mitigates risks:** R6 (see [risk-register](../03-views-and-perspectives/risk-register.md)).
