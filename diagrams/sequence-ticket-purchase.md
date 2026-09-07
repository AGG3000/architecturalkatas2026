# Sequence — Ticket purchase

**What it shows:** the ticket sale flow (including a family pass) — from selection and SSO to transactional confirmation via an external PSP and QR issuance.

The ticket-sale scenario (including a family pass). Key characteristics of Q1: **Security (PCI)** and **transactional consistency** — the only place with strong consistency. The estate does not store card data: payment is processed by an external PSP.

```mermaid
sequenceDiagram
    autonumber
    actor V as Visitor
    participant BFF as BFF / API Gateway
    participant Q8 as Q8 Identity (IdP)
    participant Q1 as Q1 Ticketing and Payments
    participant PSP as PSP (external)
    participant BB as Event Backbone
    participant Q7 as Q7 Notifications

    V->>BFF: select ticket / family pass, pay
    BFF->>Q8: verify session / token (SSO)
    Q8-->>BFF: identity OK
    BFF->>Q1: create order
    Q1->>Q1: reserve ticket,<br/>create order in PENDING status
    Q1->>PSP: initiate payment (cards not stored)
    Note over Q1,PSP: PCI isolation: card data<br/>goes directly to the PSP
    PSP-->>Q1: payment authorized (callback / webhook)
    Q1->>Q1: transactionally: order CONFIRMED,<br/>issue ticket / QR
    Q1-)BB: event TicketPurchased / FamilyPassIssued
    Q1-->>BFF: confirmation + ticket (QR)
    BFF-->>V: "ticket purchased" screen + QR

    BB-)Q7: TicketPurchased
    Q7-)V: push "ticket ready" (FCM/APNs)
    Note over BB: the ticket is also cached on Q2 (gate)<br/>for offline validation at entry
```

## Legend

| Element | Meaning |
|---|---|
| Actor (human) | Visitor |
| Rectangle | Quantum / external system (participant) |
| Solid arrow `->>` | Synchronous call (REST) |
| Dashed arrow `-->>` | Synchronous response |
| Arrow `-)` | Asynchronous event (into the backbone / push) |
| Note | Architectural clarification (PCI, offline) |

## Flow description

1. The visitor selects a ticket and pays via the BFF.
2. The BFF verifies identity via Q8 (external IdP / SSO).
3. Q1 creates an order in PENDING status and reserves the ticket.
4. Q1 initiates payment in the **PSP** — card data goes directly to the PSP (PCI isolation), the estate does not store it.
5. After payment authorization Q1 **transactionally** moves the order to CONFIRMED and issues the ticket/QR (strong consistency is exactly here).
6. Q1 publishes the `TicketPurchased` / `FamilyPassIssued` event to the backbone.
7. The BFF returns the confirmation and QR to the visitor.
8. Q7 sends a push on the event; the ticket is also cached on Q2 (gate) for offline validation at entry under patchy Wi-Fi.
