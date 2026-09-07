# Data flow — Data → Forecast → Action (Crowd Prediction)

**What it shows:** the predictive loop — from measuring queues to preemptive actions (route, staff, pushes). It illustrates that we do not just measure but **forecast and act**.

```mermaid
flowchart LR
    cams["Edge AI Platform<br/>(Queue Monitoring / Crowd Analytics)"]
    q3["Q3 Analytics<br/>(read-side / heatmap)"]
    fs[("Feature Store<br/>history + events + weather")]
    cp["Crowd Prediction (AIP)<br/>forecasting queues ahead"]
    q6["Q6 Route-Quest<br/>bypassing future peaks"]
    q12["Q12 Scheduling<br/>staff in advance"]
    q7["Q7 Notifications<br/>push 'queue will grow soon'"]
    app["Visitor / staff"]

    cams -->|"queue length / density"| q3
    q3 --> fs
    fs --> cp
    cp -->|"zone load forecast"| q6
    cp -->|"demand forecast"| q12
    cp -->|"peak alert"| q7
    q6 --> app
    q7 --> app
    q6 -.->|"price weights (Dynamic Pricing)"| q12
```

## Legend
Solid — data flow/signal · dashed — secondary link · `(...)` — storage/features.

## Key points
- The source is **fixed cameras (Queue Monitoring)**, not only app telemetry (ADR-011/020).
- The **forward** forecast (not measuring the fact) feeds three actions at once: route (Q6), schedule (Q12), notification (Q7).
- Mitigation of a self-fulfilling forecast — smoothing/soft distribution (ADR-020).
- The same demand forecast feeds Dynamic Pricing (ADR-021).
