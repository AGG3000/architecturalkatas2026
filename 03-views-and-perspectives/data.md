# Data view

## Information model (key entities)

A logical model on top of database-per-quantum: entities physically live in their own quanta and are linked between quanta via events and identifiers (not via shared FKs). (the model is an assumption on top of the brief's facts)

| Entity | Owner | Key attributes | Relationships |
|---|---|---|---|
| **Visitor / Ticket** | Q1 (+ Q8 identity) | ticket_id (pseudonym), type (regular/family pass), status, validity, order_id; identity — only in Q8 on opt-in | Ticket → VisitEvent (by ticket_id); Ticket → Visitor (in Q8) |
| **Animal** | Q5 | animal_id, species, enclosure, nutrition/health profile, for piranhas — a population tag | Animal → Enclosure; Animal → TelemetryReading; Animal → HealthEvent |
| **Enclosure** | **Q4** (Q5 — read-only via events) | enclosure_id, type (aquatic/terrestrial), environmental thresholds, criticality/nocturnal | Enclosure → Animal (1..N); Enclosure → TelemetryReading |
| **TelemetryReading** | Q4 (time-series) | reading_id, source (sensor/camera metadata), type (weight/movement/t°/environment), value, timestamp, enclosure_id/animal_id | → Animal/Enclosure; aggregates → Q3 warehouse |
| **HealthEvent** | Q5 | event_id, animal_id, type (anomaly/alert/feeding), model confidence, keeper confirmation status | ← TelemetryReading + CV; → Notification (Q7) |
| **VisitEvent** | Q3 (read-side) | event_id, ticket_id (pseudonym), type (ZoneEntered/RideBoarded/DwellTime/Purchased), zone_id, timestamp | ← Ticket; aggregated into heatmap/dwell |
| **ContentClip** | Q11 | clip_id, source (Q5 camera / Q10 360-tour), moment type, status in the approval queue, posting platforms | ← HealthEvent/highlight; a human approves |
| **Schedule** | Q12 | schedule_id, type (rostering/feeding/maintenance), resource, window, hard constraints, approval status | ← Q3 forecast + Q5 needs; → Q7/Q11/Q6 |

Principle: between quanta — **event exchange and reference by id**, not a shared schema; ticket_id travels as a pseudonym (see Privacy). **Single-ownership:** each entity has one owning quantum for its schema (e.g., `Enclosure` — Q4; Q5 reads via events) — under the MVP merge of Q4+Q5 this removes dual ownership and the distributed-monolith risk.

## Data ownership (database-per-quantum)
- Q1 Orders/Payments DB (isolated, PCI) · Q3 Analytics warehouse (read-side) · Q4 time-series (telemetry) · Q5 inference results · Q8 Auth store · Q9 Feedback · Q10 on-board buffer + transport state · Q11 content/asset store + approval-queue · Q12 schedules · AIP prompt/version registry + eval logs.

## Lakehouse & Feature Store (analytics/ML)
On top of database-per-quantum — a single analytics/ML platform (see [ADR-022](../04-adrs/ADR-022-lakehouse-and-feature-store.md)):
- **Lakehouse** (medallion bronze→silver→gold, batch+streaming) is fed from the quanta's operational DBs via a **CDC/event stream** (reusing the bus) — the quanta's operational autonomy is not broken.
- **Feature Store** — unified online/offline features for Crowd Prediction (ADR-020), Dynamic Pricing (ADR-021), Animal Health (Q5), return forecast (Q6).
- **Semantic Layer** — defined business metrics (attendance, revenue, zone load, welfare indicators) on top of the gold layer: it provides **consistent metrics** and **grounding for the Management Agent's NL analytics** (text-to-analytics over defined metrics, not over raw tables → fewer hallucinations).
- Separation of source-of-truth: **operations** — in the quanta's DBs; **analytics/ML** — in the lakehouse.
- **Lineage + consent-for-training** — at the lakehouse level (governance, ADR-018).

**Digital Twin (conceptually, not a separate component).** The park's "digital twin" is represented by a **set of current domain projections and aggregates** (gold layer + Semantic Layer) on top of the Lakehouse data — a live representation of the state of assets, visitors, queues, staff and animals. AI agents use these projections as a single context for decisions. We do **not** introduce a separate Twin component/layer: the twin is a projection of already existing data, not a new system (near-real-time context for agent dialogue is the fast memory tier, see ADR-024).

## Flows and consistency
- Backbone: **two kinds of topics** — **compacted** (snapshot, "latest state by key": statuses, positions, configs) and **retained-as-log** (full history for audit/training: `HealthEvent` 2–3 yrs, `TicketPurchased`, telemetry). Compaction applies to state topics only, **not** to the event log.
- A single **partitioning key** for paired topic↔table (ordering of updates).
- **Eventual consistency** by default; strict — only Q1 (payments).
- Reliability: **Inbox/Outbox**, at-least-once + idempotency by event id.

## Event contracts and their evolution
The bus is the **API between quanta**, so an event contract is managed as an API:
- **Schema registry** — each event has a versioned schema; the consumer validates the source and the schema (see [security-privacy](security-privacy.md)).
- **Compatibility policy** — changes are **backward/forward-compatible** (adding optional fields); removal/renaming — via a new major version + a dual-publication period.
- **Contract owner** — each canonical event (`TicketPurchased`, `AnimalHealthAlertRaised`, `TelemetryReading`) has an **owning quantum**; consumers (Q7/Q2/Q3/lakehouse) do not change it.
- **Consumer-driven contract tests** in CI — the producer does not break consumers; an incompatible change = a red gate.
- **Partition key — by the relevant aggregate:** `TelemetryReading` is partitioned by `animal_id` for the ordering of welfare events per animal (not by `enclosure_id`); the key of the topic↔table pair is fixed in the contract.
- Runtime — discarding of stale versions by number ([ADR-007](../04-adrs/ADR-007-event-reliability-inbox-outbox.md)).

## Lifecycle
- **Animal telemetry (Q4 time-series):** raw data — a "hot" window of **90 days** at full resolution; then **downsampling** (hour/day aggregates) and transfer to the warehouse (Q3) for **≈2 years** for health/seasonality trends; raw camera video is **not stored** (edge inference, only events go outward; optionally a short local buffer for investigations — hours/days). Example thresholds, tuned by cost. (assumption)
- **HealthEvent/alerts (Q5):** stored longer than telemetry (**≈2–3 years**) as an audit of welfare decisions and a training signal.
- **VisitEvent (Q3):** individual events under a pseudonymous ticket_id — a short retention, then only **de-identified aggregates** (heatmap/dwell) for a long period; the personal raw history is deleted together with the ticket_id key.
- **Mobile telemetry:** pseudonymous ticket ID → **crypto-shredding** on completion of the visit (deleting the key makes the history unrecoverable).
- **Edge buffer (data mule / store-and-forward):** deleted after a confirmed idempotent upload to the cloud.

## Storage sizing
Orders of magnitude (assumption — the brief gives no volumes; calibrate on real sensors/traffic). **Raw stays on the edge** (video, vibration waveforms) → the analytical lake ingests events/metadata/features, so it stays small. Three separate stores with different retention:

### 1. Analytical lake (Lakehouse, Parquet/Iceberg)
| Source | ~GB/yr |
|---|---|
| IoT climate/telemetry (compressed; 90-day hot + downsample) | ~5 |
| VisitEvent Q3 (individual short → aggregates) | ~10 |
| Ride PdM **features** (Q13 — not waveforms) | ~3 |
| CV metadata (heatmap/queues/welfare events) | ~5 |
| Health/audit + AI-decision audit + eval logs | ~4 |
| Feature Store (derived) | ~2 |

Bronze ~30 → medallion (×~1.8) ≈ **~50 GB/yr → ~100–150 GB over 3 yr**. Object storage ≈ $0.02/GB/mo → **≈ $2–4/mo**. The Lakehouse cost line is mostly **compute/query/feature-serving, not bytes**.

### 2. Media (object storage / CDN — separate)
360 tours + highlight clips: raw 360 ≈ 1–1.5 TB/yr (≈ $20–35/mo); with **"keep highlights, prune raw after processing"** → ~50–200 GB/yr (≈ $1–5/mo).

### 3. Observability & audit (separate stack, short retention, sampled)
| Stream | ~volume (hot) |
|---|---|
| Metrics (SLO/latency/fitness/PSI/cost) | a few GB/yr (priced by series) |
| Logs (services/edge/ingest; 7–14-day hot, then archive/drop) | ~30–100 GB hot |
| **Agent traces** (span per tool-call; **sampled**: 100% errors + a fraction of successes) | ~15–35 GB/yr |
| AI-decision audit (by **reference**, not payload) | ~10–40 GB/yr; welfare subset 2–3 yr |

≈ 50–200 GB/yr hot ≈ **$50–150/mo** (in the "IdP · maps · observability" + AIP-audit lines).

**Levers & privacy.** Trace sampling; log levels + short hot retention; controlled metric cardinality; audit by reference, not payload. **Edge logs are delay-tolerant** — shipped via the same store-and-forward/mule, not the real-time channel. **Agent traces carry prompts/responses/retrieved context → potential visitor PII:** PII-redaction on ingest, limited trace retention, role-based access ([security-privacy](security-privacy.md)) — observability must not become a shadow copy of personal data.
