# Sequence — Route-quest step

**What it shows:** the flow of one step of the dynamic route-quest — requesting next-best-step from Q6 and optimizing the path across 3 goals with queue data and commercial weights.

The scenario of one step of the dynamic route-quest: the app requests **next-best-step** from **Q6**, which optimizes the path across 3 goals (minimum queues × maximum experience × commercial benefit). Queue/heatmap data comes from **Q3**, commercial weights and the reward — via **Q1**. Inference/generation — via the AIP.

```mermaid
sequenceDiagram
    autonumber
    actor V as Visitor (PWA)
    participant BFF as BFF (mobile)
    participant Q6 as Q6 Growth / Route-Quest
    participant BB as Event Backbone
    participant Q3 as Q3 Movement and Analytics
    participant Q1 as Q1 Ticketing (commercial weights)
    participant AIP as AIP (LLM / optimization)

    Note over V: on-device: map cache, proximity calc,<br/>store-and-forward telemetry (ticket ID)
    V->>BFF: request next-best-step (current position, ticket ID)
    BFF->>Q6: next-best-step

    Q6->>Q3: live load of zones / queue lengths / heatmap
    Note over Q3: queues from fixed cameras (edge-CV),<br/>dwell-time from app telemetry
    Q3-->>Q6: queues / zone density (real-time)
    Q6->>Q1: commercial weights / promo / available reward
    Q1-->>Q6: weights + reward pool (curated)

    Q6->>AIP: path optimization + AI quest generation<br/>(narrative / challenge, guardrails)
    AIP-->>Q6: next step + storytelling
    Q6-->>BFF: next step (where to go, reward for completion)
    BFF-->>V: show the quest step in the app

    V-)BFF: checkpoint passed (batched telemetry)
    BFF-)BB: ZoneEntered / QuestStepCompleted (ticket ID)
    BB-)Q3: movement events (analytics)
    BB-)Q1: credit reward (ticket / discount)
    Q1-)V: reward (push via Q7)
```

## Legend

| Element | Meaning |
|---|---|
| Actor (human) | Visitor via PWA |
| Rectangle | Quantum / BFF / AIP (participant) |
| Solid arrow `->>` | Synchronous call (REST) |
| Dashed `-->>` | Synchronous response |
| Arrow `-)` | Asynchronous event / reward via the backbone |
| Note | Architectural context (on-device, data sources) |

## Flow description

1. The app (PWA) computes much **on-device** (map cache, proximity, store-and-forward of telemetry under a pseudonymous ticket ID) and requests next-best-step from the BFF.
2. The BFF calls **Q6** (the route-quest optimization engine).
3. Q6 takes the live load of zones and queue lengths from **Q3** (queues — from fixed-camera edge-CV; dwell-time — from app telemetry by ticket ID).
4. Q6 requests from **Q1** the commercial weights, promo, and the available (curated) reward.
5. Q6 optimizes the path across 3 goals via the **AIP** and generates the quest narrative/challenge (with guardrails).
6. The next step with the reward is returned to the app.
7. Passing a checkpoint goes in batches as events (`ZoneEntered` / `QuestStepCompleted`) into the backbone → to Q3 (analytics) and Q1 (reward crediting — ticket/discount, push via Q7).
