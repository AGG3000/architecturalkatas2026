# How we used AI (building this architecture)

The kata's theme is *AI-Assisted Software Architecture* — so, under human direction and with the team accountable for every decision, we also used AI in the **process** of designing it (not only inside the solution).

## Where AI helped
- **Research & synthesis** — surveying patterns and standards (C4, event-driven/CQRS, evolutionary architecture, EU AI Act) against the brief's constraints.
- **ADR drafting** — first drafts of Context/Decision/Alternatives, then human trade-off review and sign-off.
- **Diagram generation** — Mermaid diagrams from prose; humans fixed the semantics, phasing and layout.
- **Adversarial review** — several independent AI review "lenses" (quality/V&V, AI-platform, DDD/EDA) stress-tested the design and surfaced gaps we then closed.

## Where AI was wrong — and how we caught it
- **Thermal cameras for cold-blooded animals** — physically wrong (thermal only sees warm bodies). Caught in review → RGB-CV + breach/RFID for cold-blooded species; thermal only for warm-blooded / fire / ride overheating / intruders (see [ADR-011](04-adrs/ADR-011-fixed-cameras-edge-cv-vs-cloud-cv.md)).
- **Phasing drift** — the data-mule appeared as MVP while the shuttle is roadmap R3. Caught by consistency checks → MVP connectivity = radio-bridge/cellular + store-and-forward; the mule arrives with the shuttle in R3.
- **Scope-count drift** — the MVP quantum count was inconsistent across files. Caught → one definition (7 capabilities = 10 of 13 quanta).

## Guardrails on our own process
- **Human sign-off on every ADR and trade-off** — AI drafts, humans decide.
- **Fact-vs-assumption discipline** — a single source of truth for brief facts; every assumption marked "(assumption)".
- **No fabricated certainty** — cost and OKR figures are labeled assumptions, not facts.

*All architectural decisions, trade-offs and final content were made, reviewed and approved by the team, who take full responsibility for them.*
