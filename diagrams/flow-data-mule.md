# Data flow — Data-mule (delay-tolerant under patchy Wi-Fi)

**What it shows:** how data from a remote enclosure (out of stable connectivity) reaches the cloud via the shuttle "mule" — pickup on drive-by, buffering, upload at charging. It illustrates operation under **patchy Wi-Fi** (brief constraint §E).

```mermaid
sequenceDiagram
    autonumber
    participant ENC as Enclosure (out of connectivity)<br/>Edge AI Platform + buffer
    participant SH as Shuttle (mobile edge)<br/>collector + buffer
    participant DEPOT as Depot / charging<br/>(wired link ≈1 Gbit/s)
    participant ING as Cloud ingest
    participant LAKE as Lakehouse / Twin

    Note over ENC: delay-tolerant data accumulates<br/>(metadata, rare clips). Real-time → cellular (bypassing the mule)
    SH->>ENC: drive-by (BLE/Wi-Fi contact)
    ENC-->>SH: buffer upload (metadata KB ≈5s, clips MB ≈1s)
    Note over SH: on-board buffer + 360 recording accumulate
    SH->>DEPOT: arrival at charging (standard dwell 10–30 min)
    SH->>ING: dump: metadata (seconds) + 360 (≈2–4 GB, minutes)
    ING->>ING: idempotently (at-least-once + dedup by event id)
    ING->>LAKE: CDC/stream into medallion + feature store
    Note over LAKE: data catches up with a delay of minutes to tens of minutes (acceptable)
```

## Legend
Solid — synchronous exchange/upload · dashed (`-->>`) — response/data transfer · Note — principle/constraint.

## Key points
- **Real-time-critical does not ride the mule** — it goes over cellular/fixed channel (ADR-004/017).
- Pickup — seconds (metadata) to ≈sec (clips); the heavy 360 is dumped during **standard charging** (ADR-026).
- Idempotency guarantees no duplicates under at-least-once (ADR-007).
- For enclosures off the route — a separate utility mule cart (ADR-004).
