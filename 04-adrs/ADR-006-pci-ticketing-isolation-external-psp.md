# ADR-006: PCI isolation of Ticketing and an external PSP (we do not store cards)

## Status
Accepted

## Context
A ticket-purchasing system is needed, including family passes (brief fact, §D1), with revenue growth as a business goal (brief fact, §D4). Payments require Security and transactional consistency — characteristics that diverge from the offline/read-side of the other quanta (see [ADR-002](ADR-002-quantum-boundaries-by-characteristics.md)). Accepting and storing cards brings the entire system under the full scope of PCI-DSS, which is expensive and risky for a recently loss-making estate with a limited budget (assumption). Payment processing is not a differentiating value of the estate (assumption).

## Decision
Carve out **Q1 Ticketing & Payments into a separate quantum with an isolated Orders/Payments database** and delegate the payment itself to an **external PSP** (Stripe/Adyen class): card data is entered into the PSP widget/tokenized, **we neither store nor process cards** — we keep only tokens/references to transactions. The PCI scope collapses to the integration with the PSP. Rationale: transfer of the compliance risk to the vendor + isolation of the security quarter.

## Consequences
- Pros: minimal PCI scope (SAQ-A class), cards not stored, risk/compliance on the PSP, isolation of the secure domain from the rest of the system, readiness for family passes/refunds.
- Cons: dependency on the PSP (availability, fees ~2.9%+$0.30/transaction), less control over the payment UX, vendor lock-in at the integration level.
- Mitigations: payments abstracted behind an interface (option of a second PSP), at-least-once + idempotency against double charges (see [ADR-007](ADR-007-event-reliability-inbox-outbox.md)), strong consistency within Q1 (an exception to eventual-by-default, [ADR-001](ADR-001-event-driven-architecture-and-quanta.md)).
- **Partial failure of a distributed payment** ("PSP charged, but Q1 did not confirm" / a lost webhook): the PSP is the **source of truth** for the payment; an idempotent **reconciliation job** reconciles webhook ↔ order and **compensates** (auto-retry of confirmation or auto-refund); the webhook is processed idempotently (dedup by event id). This is a compensating pattern instead of a distributed transaction.

## Considered / Rejected alternatives
- **Store/process cards ourselves** — rejected: full PCI-DSS scope over the whole system, expensive and risky for the estate's budget, gives no business advantage.
- **Merge Ticketing with the other quanta** — rejected: PCI and consistency would "leak" into the offline/analytics quanta, imposing extra security requirements on them and killing failure isolation (contradicts [ADR-002](ADR-002-quantum-boundaries-by-characteristics.md)).

## Traceability
Brief goal "ticket purchase + family passes + profit growth" → characteristics Security(PCI)·Consistency → isolation of the Ticketing quantum + external PSP without storing cards. Related ADRs: [ADR-002](ADR-002-quantum-boundaries-by-characteristics.md), [ADR-007](ADR-007-event-reliability-inbox-outbox.md).

**Mitigates risks:** R15, R17 (see [risk-register](../03-views-and-perspectives/risk-register.md)).
