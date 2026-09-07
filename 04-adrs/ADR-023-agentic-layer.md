# ADR-023: Agentic Layer (tool-using AI agents) on top of the AIP

## Status
Accepted

## Context
Theme §G (AI-assisted); the brief directly shows an agent example (a Holiday Agent via MCP). We have RAG assistants ([ADR-019](ADR-019-rag-knowledge-assistant.md)) and predictors, but no **tool-using agents** that perform **multi-step tasks through tools**. Agents are introduced **by four stakeholder groups** (visitors/staff/veterinarians/management, see `../05-ai/agents.md`): first — **Visitor Concierge** and **Operations Copilot** (MVP), then the **Animal** and **Management** agents (roadmap R2). An agent that takes actions is safety/governance-sensitive (tool-misuse, prompt-injection, unpredictable chains).

## Decision
Introduce an **agentic layer on top of the AIP**: an agent = an LLM + **strictly typed tools** (tool-calling / MCP-class) with orchestration. Agents reach services **only through typed tools with least-privilege**, not directly to the DB. Primary agents (**MVP**):
1. **Visitor Concierge Agent** (visitors) — tools: route (Q6), RAG info ([ADR-019](ADR-019-rag-knowledge-assistant.md)), ticket purchase (Q1), pushes (Q7).
2. **Operations Copilot** (staff) — tools: analytics query (Q3), schedule draft (Q12), maintenance initiation, SOP answers (RAG).

**Roadmap agents (R2, the same pattern — typed tools + HITL + two-tier memory [ADR-024](ADR-024-lakehouse-as-agent-memory.md)):**
3. **Animal Agent** (veterinarians) — tools: telemetry/anomalies (Q5), recording a welfare outcome, escalation to the keeper (Q7); welfare interventions are governance-class **high**, veterinarian confirmation is mandatory.
4. **Management Agent** (management) — tools: NL analytics (Q3, read-only), pricing-policy approval (Q1/[ADR-021](ADR-021-dynamic-pricing-ai.md)), summaries/reports, content approval (Q11).

Any **action with an effect** (purchase, schedule change, maintenance start) goes **through human-in-the-loop / approval** and permissions; tools are **idempotent**, actions are **logged** (audit). The autonomous shuttle (Q10) is a separate physical agent, already under ODD/fail-safe ([ADR-015](ADR-015-autonomous-shuttle-and-data-mule.md)).

## Consequences
- Pros: multi-step, real "AI-assisted" value; a unified tool pattern; reuses RAG/forecasts/services; answers the trend and the brief's example.
- Cons: agency raises risk (unpredictable chains, tool-misuse, prompt-injection); cost; the complexity of orchestration and testing nondeterminism.
- Mitigations: typed tools + least-privilege; **human confirmation of actions with effects**; guardrails and PI protection; a limited tool whitelist; governance-class medium/high ([ADR-018](ADR-018-ai-governance-framework.md)); **idempotency of tool calls** via Outbox/business key ([ADR-007](ADR-007-event-reliability-inbox-outbox.md)) — a retry is harmless.
- **Quality monitoring (in prod):** AI quality is continuously tracked by **Task Success Rate**, **Tool Error Rate**, and **Human Override Rate**; a noticeable rise in human overrides is an early indicator of degradation. When thresholds are exceeded, the platform auto-triggers mitigation: model/prompt rollback, prompt review, strengthening human-approval, narrowing the allowed actions (see [agents.md](../05-ai/agents.md), [validation-verification.md](../05-ai/validation-verification.md)).

## Considered / Rejected alternatives
- **Direct agent access to the DB/services without typed tools** — rejected: unsafe (bypassing permissions, tool-misuse), not auditable.
- **Fully autonomous agents acting without a human** — rejected: reputational/safety risk, fails V&V (as in [ADR-010](ADR-010-human-in-the-loop-confidence-thresholds.md), [ADR-016](ADR-016-ai-marketing-and-scheduling-hitl-provider-abstraction.md)).
- **Keep only assistants (no agency)** — rejected: does not answer the trend/the brief's example, we lose the multi-step value.

## Traceability
§G (AI-assisted) + the brief's example (agent/MCP) → Evolvability + Governance/Safety → an agentic layer on typed tools with human-approval. Details — `../05-ai/agents.md`. Related ADRs: [ADR-008](ADR-008-ai-platform-provider-abstraction-fallback-cost.md), [ADR-010](ADR-010-human-in-the-loop-confidence-thresholds.md), [ADR-015](ADR-015-autonomous-shuttle-and-data-mule.md), [ADR-018](ADR-018-ai-governance-framework.md), [ADR-019](ADR-019-rag-knowledge-assistant.md).

**Mitigates risks:** R9 (see [risk-register](../03-views-and-perspectives/risk-register.md)).
