# Operational & team effort perspective

Not an exact payroll (salaries are out of scope; the brief does not set the team size → **assumption**), but an estimate of the **load on teams and support**: how many teams/roles are needed, how build-vs-buy and evolution reduce headcount, and what the ops load of the edge devices is.

## Principles that reduce the load
- **Team-sized quanta** — a quantum is held by one small team (see the MVP roll-up in `../02-solution/mvp-vs-roadmap.md`).
- **Build vs Buy** — we buy the non-core (IdP/Q8, PSP, social API, chat) → no separate teams for them ([ADR-006](../04-adrs/ADR-006-pci-ticketing-isolation-external-psp.md), [ADR-016](../04-adrs/ADR-016-ai-marketing-and-scheduling-hitl-provider-abstraction.md)).
- **Start with modular monoliths** → a small team does not carry distributed complexity prematurely; split into services on a load signal.
- **Provider abstraction (AIP)** → switching an AI model = configuration, not a rewrite project.
- **Engineering practices** (CI/CD, observability, feature toggles, IaC) → less manual maintenance.

## Estimate of teams/roles (assumption)

| Role / team | MVP (~5000/day) | Scale (~15000/day) |
|---|---|---|
| Platform/Backend (Q1/Q3/Q7/Q8 + AIP, modular monolith) | 1 small team | +split into 2 teams |
| AI/Data (Q5, Q3 analytics, models behind the AIP) | 1–2 engineers | +1 (Q11/Q12 AI) |
| Frontend/Mobile (PWA) | 1 engineer | 1–2 |
| Edge/IoT technician (cameras, MQTT sensors, devices) — field support | part-time / contract | 1 dedicated |
| SRE/Ops (observability, incidents) | shared | 1 |
| Transport/autonomy (Q10) | — (roadmap phase; vendor-managed) | vendor + integration |

## Operational (non-dev) roles from human-in-the-loop
AI is designed with a human in the loop → this is an **operational load on the estate's existing staff**, not new dev teams:
- **keepers** confirm welfare alerts (Q5) and local emergency triggers;
- a **content curator** approves publications (Q11);
- a **manager** approves schedules (Q12);
- a **teleoperator** — only once the autonomous shuttle reaches its roadmap phase (Q10).

## Edge ops load (case specifics)
Physical devices require their own support (not only the cloud):
- field maintenance of cameras/sensors/gate controllers;
- **OTA model updates** on the edge (managed via the AIP);
- provisioning/rotation of device certificates, data-mule reliability (shuttle);
- connectivity degradation/backup — see `reliability.md`.

## Conclusion
The architecture **deliberately minimizes headcount**: buy the non-core, team-sized quanta, monolith at the start, provider abstraction. The main additional load relative to a purely cloud solution is **field edge support** and the **human-in-the-loop operational roles**, which mostly fall on the estate's already existing staff (keepers/managers), not on hiring new developers. Exact payroll — out of scope (assumption).
