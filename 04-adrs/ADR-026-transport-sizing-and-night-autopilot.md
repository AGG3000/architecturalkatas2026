# ADR-026: Transport — passenger model, data-mule throughput, and night autopilot trips

## Status
Accepted

## Context
The size and type of the internal transport (Q10) had not until now been justified by passenger flow — yet that is the main driver (the data the data-mule carries is light). The ≈100-acre territory is walkable → transport = convenience/premium/access to remote zones, not mass transit. Separately: the asset **sits idle at night**, while many exotic/venomous animals are **nocturnal** — their telemetry must be captured precisely at night. Night (the park closed, no crowds/children) is the **lowest-risk window** for autonomous driving.

## Decision
1. **The fleet is scaled by passengers, the type is a land-train (trackless tram, ≈72 seats)**, not a 12-seat shuttle. Model: `boardings/day = visits × transport_share × 2`; peak/hour = day/10h × 1.5. Consist throughput ≈ 72 × 3 cycles/h × 0.8 ≈ ≈170 people/h; round-trip ≈20 min.
   | Scenario | Visits | Share (assumption) | Peak people/h | Consists (+reserve) |
   |---|---|---|---|---|
   | MIN | 5,000 | 15% | ≈225 | ≈3 |
   | PROJECTED | 15,000 | 20% | ≈900 | ≈6 |
   | RAPID | 25,000 | 25% | ≈1,875 | ≈12 (or higher-capacity consists) |
2. **The data-mule is an incidental function** (free-rider), we do not build a separate data-only fleet. Volume: ≈**50 MB/loop** pickup from enclosures (metadata + rare clips) + a **360 recording ≈2–4 GB/loop** (generated on board). Pickup: KB → LoRa/BLE <5 s (on the pass), MB → short-range Wi-Fi ≈1 s; dump: metadata — seconds, 360 — during **scheduled charging** (a fast wired dock ≈1 Gbit/s, 10–30 min). For enclosures **off** the passenger route — a cheap **utilitarian mule cart** (see [ADR-004](ADR-004-edge-connectivity-mesh-cellular-data-mule.md)).
3. **On-board devices:** an edge-collector + an SSD buffer (256 GB–1 TB, encryption, tamper-resistance); multi-radio (Wi-Fi AP/BLE/LoRa + a **dual-carrier cellular** for real-time-bypass); GNSS; optionally a 360 camera, burrow sensors, an info board, edge-CV.
4. **Night autopilot trips for metrics (driverless, on autopilot).** At night (the park closed, no visitors) the consist runs **driverless on autopilot** along the routes and: captures **nocturnal-animal telemetry** (a data-mule sweep — critical, the collection is predominantly nocturnal), runs a **thermal-imaging security patrol of the perimeter/intruders**, environmental measurements, a drive-by infrastructure check/360-mapping. Night = the **minimal-risk window** → this is the **first phase of the transition to driverless** in the autonomy roadmap (supervised by day → driverless by night → onward).

## Consequences
- Pros: the fleet is justified by passenger flow; 24/7 asset utilization; **fresh nocturnal welfare data** without a passenger schedule; a night security patrol; a **safe start of driverless** (no crowds/children).
- Cons: the night autopilot still requires a safety envelope (geo-fencing, low speed, fail-safe, teleoperation oversight); coordination with charging/maintenance windows; a risk of encountering an escaped animal on the way (fail-safe to stop).
- Mitigation: an ODD for the night route, teleoperation, fail-safe; night reduces, but does not remove, the safety requirements (see [ADR-015](ADR-015-autonomous-shuttle-and-data-mule.md)).

## Considered / Rejected alternatives
- **A separate data-only fleet** — rejected: underutilized; the data is light and "rides for free" on the passenger consist.
- **12-seat shuttles instead of a land-train** — rejected: at PROJECTED ≈900 people/h it would require an unjustifiably large fleet.
- **Night trips with a driver** — rejected: operating costs; a night without crowds is exactly the safe place for driverless.
- **No night trips at all** — rejected: we lose the nocturnal telemetry (nocturnal animals), utilization, and the safe window for shaking down autonomy.

## Traceability
§C (scale 5000→15000, territory) + the profitability goal + §D (animal monitoring, incl. nocturnal) → **Availability/Cost/Safety** → a fleet by passengers (land-train), the data-mule incidentally, night driverless metric collection as the first phase of autonomy. Related ADRs: [ADR-004](ADR-004-edge-connectivity-mesh-cellular-data-mule.md), [ADR-015](ADR-015-autonomous-shuttle-and-data-mule.md), [ADR-020](ADR-020-crowd-prediction-ai.md).
**Mitigates risks:** R3 (freshness/delivery of data, incl. nocturnal) (see [risk-register](../03-views-and-perspectives/risk-register.md)).
