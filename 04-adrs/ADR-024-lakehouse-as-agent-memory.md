# ADR-024: Lakehouse as shared agent memory + a working-memory tier

## Status
Accepted

## Context
The solution is built around per-stakeholder agents (Visitor/Operations/Animal/Management, [ADR-023](ADR-023-agentic-layer.md)) that need **shared context and institutional memory** — the history of visits, animal health, decisions, patterns. The temptation is to make "Lakehouse = agent memory". But the Lakehouse ([ADR-022](ADR-022-lakehouse-and-feature-store.md)) is batch/medallion, eventual-consistency, optimized for analytics/ML, **not for real-time dialogue**: pulling the agent's working context from it → latency and staleness. Meanwhile, the operational source-of-truth must stay in the quanta ([ADR-002](ADR-002-quantum-boundaries-by-characteristics.md)).

## Decision
Split agent memory into **two tiers**:
1. **Long-term / institutional memory + features → Lakehouse + Feature Store** ([ADR-022](ADR-022-lakehouse-and-feature-store.md)): history of visits/health/decisions, patterns, training data, lineage. Shared across all agents.
2. **Working (short-term) memory → a fast tier:** a vector KB (RAG, [ADR-019](ADR-019-rag-knowledge-assistant.md)) + a session/state store (dialogue context, recent events). Populated from the lakehouse, but lives separately for the sake of latency and freshness.

All agents **read context** from these tiers and **write back** derived "memory" (outcomes, feedback, labeling) into the lakehouse — a closed loop that improves future answers. **Operations** (ticket, schedule, intervention) are performed by agents **only through typed tools** of the services ([ADR-023](ADR-023-agentic-layer.md)), not by writing to the operational DBs directly.

**Write-back is governed, not free (closed-loop protection).** A write to memory passes a **write-guard**: only typed derivatives by schema (not instructions), provenance (agent/session/trace) and dedup by business key; critical derivatives (welfare/safety/economics) are **human-approved**. Memory is **versioned with lineage**, which makes it possible to **roll back a poisoned/degraded batch** by provenance — like a model/prompt rollback. Against loop self-reinforcement (an agent learning from its own outputs) a **write-back drift monitor** is kept (derivative PSI, share of self-generated vs human-verified). The worked memory-poisoning kill-chain — [agents.md](../05-ai/agents.md#prompt-injection--tool-misuse-trust-boundary).

## Consequences
- Pros: context and continuity for agents; low latency of working memory; unified institutional memory; a closed learning loop; operational integrity preserved.
- Cons: two memory stores complicate the system; lakehouse→working synchronization (freshness); a risk of PII leaking into memory → retention/consent policies; **the closed write loop → a risk of memory poisoning and self-reinforcement** (an agent learning from its own outputs).
- Mitigations: working-memory is populated from the lakehouse on a schedule/events; TTL and PII minimization in working memory (pseudonyms, [ADR-013](ADR-013-privacy-surveillance-and-mobile-telemetry.md)); governance/consent for writing to memory ([ADR-018](ADR-018-ai-governance-framework.md)); **write-guard (schema + provenance + dedup) + human-approved for critical derivatives; memory versioning/lineage for batch rollback; write-back drift monitoring (PSI) against self-reinforcement** — kill-chain in [agents.md](../05-ai/agents.md#prompt-injection--tool-misuse-trust-boundary).

## Considered / Rejected alternatives
- **Lakehouse as the sole agent memory** — rejected: batch/eventual is not suitable for real-time dialogue (latency/staleness).
- **Only working-memory without a lakehouse** — rejected: no institutional memory, history, features, or lineage; agents are "forgetful".
- **Agents writing directly to the operational DBs** — rejected: it wrecks quanta autonomy and operational integrity; actions go only through tools.
- **Free-form (unstructured) writes to memory without a write-guard** — rejected: opens memory poisoning and silent self-reinforcement, breaks rollback (no provenance/schema); writes are limited to typed derivatives through the guard.

## Traceability
§G (AI-assisted in business processes) → agents with shared memory → two-tier memory (lakehouse long-term + fast working) with the operational SoT in the quanta. Related ADRs: [ADR-019](ADR-019-rag-knowledge-assistant.md), [ADR-022](ADR-022-lakehouse-and-feature-store.md), [ADR-023](ADR-023-agentic-layer.md), [ADR-002](ADR-002-quantum-boundaries-by-characteristics.md), [ADR-018](ADR-018-ai-governance-framework.md).

**Mitigates risks:** R8 (see [risk-register](../03-views-and-perspectives/risk-register.md)).
