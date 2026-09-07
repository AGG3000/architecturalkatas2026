# Stakeholders

Who uses the solution, what their goals are — and **how the architecture addresses them** (mapping to quanta/ADRs = bottom-up traceability). AI directly helps **four groups through agents** (visitors / staff / veterinarians / management, see [agents](../05-ai/agents.md)); below is the full map of personas and groups.

## Personas (5)

| Persona | Role | What matters | Who/what serves them |
|---|---|---|---|
| **Margaret** | Estate Owner (72nd Countess) | growth and profitability, strategic decisions, sustainability | Management Agent · NL analytics (Q3) · Dynamic Pricing (Q1) |
| **Oliver** | Park Operations Manager | staff allocation, uninterrupted attractions, incidents, costs | Operations Copilot · schedules (Q12) · attraction status (Q13) · alerts (Q7) |
| **Emma** | Family Visitor | short queues, family pass, personalization, a safe visit | Visitor Concierge (Q6) · Crowd Prediction (Q3) · Ticketing/family pass (Q1) · RFID |
| **Dr. Sarah** | Chief Veterinarian | animal health, early anomaly detection, welfare | Animal Agent · Animal Health AI (Q5) · IoT (Q4) · thresholds + HITL (ADR-010) |
| **Michael** | Marketing Director | attendance, returning guests, membership sales | AI Social Marketing (Q11) · return forecast (Q6) |

## Stakeholder groups and goals

| # | Group | Goals | How the solution addresses it |
|---|---|---|---|
| 1 | **Visitors** (Families · Tourists · Season Pass · VIP) | short queues · personalization · easy purchase · a safe visit | Crowd Prediction ([ADR-020](../04-adrs/ADR-020-crowd-prediction-ai.md)) · Visitor Concierge (Q6/RAG, [ADR-019](../04-adrs/ADR-019-rag-knowledge-assistant.md)/[ADR-023](../04-adrs/ADR-023-agentic-layer.md)) · Ticketing + family passes ([ADR-006](../04-adrs/ADR-006-pci-ticketing-isolation-external-psp.md)) · RFID ([ADR-027](../04-adrs/ADR-027-visitor-rfid-token.md)) · safety (Q13/Q5) |
| 2 | **Park Operations Team** | efficient allocation · uninterrupted attractions · incident management · lower costs | AI schedules (Q12, [ADR-016](../04-adrs/ADR-016-ai-marketing-and-scheduling-hitl-provider-abstraction.md)) · attraction status (Q13) · alerts (Q7) · Operations Copilot ([ADR-023](../04-adrs/ADR-023-agentic-layer.md)) · [cost](../03-views-and-perspectives/cost.md) levers |
| 3 | **Maintenance Engineers** | minimal downtime · maintenance planning · attraction safety | **Predictive maintenance (Q13)** — vibration/temperature/current → anomalies + RUL + failure forecast ([ADR-025](../04-adrs/ADR-025-ride-attractions-management.md)) |
| 4 | **Veterinarians** | health monitoring · early anomalies · welfare | Animal Health AI (Q5) · IoT telemetry (Q4) · Animal Agent · confidence thresholds + HITL ([ADR-010](../04-adrs/ADR-010-human-in-the-loop-confidence-thresholds.md)) |
| 5 | **Zoo Operations Team** | feeding schedules · population accounting · enclosure safety | Feeding (Q12) · piranha counting (Q5 CV) · breach sensors/RFID (Q4) |
| 6 | **Marketing Team** | attendance growth · retention · membership sales | AI Social Marketing (Q11, [ADR-016](../04-adrs/ADR-016-ai-marketing-and-scheduling-hitl-provider-abstraction.md)) · return forecast (Q6) · memberships (Q1) |
| 7 | **Park Management** | revenue · satisfaction · growth **5000 → 15000/day** | revenue thesis (pricing + marketing + retention, see [overview](../00-overview.md#business-value-of-ai)) · NL analytics (Q3) · Management Agent |
| 8 | **Finance Department** | price optimization · cost control · profitability | Dynamic Pricing ([ADR-021](../04-adrs/ADR-021-dynamic-pricing-ai.md)) · AI Gateway cost-monitor ([ADR-008](../04-adrs/ADR-008-ai-platform-provider-abstraction-fallback-cost.md)) · [cost](../03-views-and-perspectives/cost.md) |
| 9 | **IT Operations Team** | platform availability · operation on an unreliable network · cybersecurity | degraded-mode + dual-carrier ([ADR-017](../04-adrs/ADR-017-connectivity-failure-degraded-operation.md)) · [reliability](../03-views-and-perspectives/reliability.md) · [security-privacy](../03-views-and-perspectives/security-privacy.md) · AI Gateway |
| 10 | **Executives / Estate Owners** | business growth · support for strategic decisions · long-term sustainability | Management Agent · "digital twin" (domain projections) · roadmap phasing ([mvp-vs-roadmap](../02-solution/mvp-vs-roadmap.md)) |

## Roll-up: 10 groups → 4 stakeholder agents

The AI agents ([ADR-023](../04-adrs/ADR-023-agentic-layer.md)) aggregate these groups:
- **Visitor Agent** → group 1 (visitors; persona Emma);
- **Operations Agent** → groups 2, 3, 5 (park operations, maintenance engineers, zoo operations; persona Oliver);
- **Animal Agent** → group 4 (veterinarians; persona Dr. Sarah);
- **Management Agent** → groups 6, 7, 8, 10 (marketing, management, finance, owners; personas Margaret, Michael).

**IT Operations (group 9)** — not a conversational agent, but the owner of the platform/reliability/security (SRE); served by infrastructure decisions (edge/degraded-mode, security), not by an agent.
