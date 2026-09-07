# ADR-011: Fixed cameras — edge-CV + placement of RGB vs Thermal

*(Combines the former ADR-011 "Edge-CV vs Cloud-CV" and ADR-012 "RGB vs Thermal".)*

## Status
Accepted

## Context
Across the territory — dozens of fixed cameras (RGB/thermal) as a sensor layer for Q5 (animal health), Q3 (queues/crowd), Q7 (alerts), Q13 (overheating of rides). Three forces shape the decision:
1. **Connectivity/volume:** Wi-Fi is patchy, "a way to deliver data to the cloud" is needed (§E); video streams are bulky — cloud inference is impossible by bandwidth, expensive by egress, risky by privacy (raw frames going outside). Some recognitions (fire, a fall) are real-time-critical.
2. **Biology:** a thermal camera sees only bodies with thermal contrast — **endotherms**. The venomous collection is predominantly **cold-blooded** (t ≈ ambient) → thermal **does not detect/measure** them.
3. **Climate:** the microclimate (air, humidity, gases, light/UV, water) is measured more accurately and cheaply by **MQTT sensors** (§E), not a thermal camera.
Affected: **offline-tolerance, Performance (latency), Cost, Privacy, Accuracy**.

## Decision
**(A) Edge-CV on a unified Edge AI Platform, not cloud inference.** Introduce an **Edge AI Platform** — a **standard edge runtime**, deployed identically on all edge nodes; which pipelines are active is set by the **node's config** (not a separate "box per zone"). Responsibilities:
- **Camera Processing** — reception/decoding of RGB/thermal, preprocessing;
- **Animal Detection** — detection/re-ID/behavior (for cold-blooded ones — + breach/RFID);
- **Crowd Analytics** — density/flows/bottlenecks;
- **Queue Monitoring** — queue length → waiting time;
- **Local Buffering** — store-and-forward: outside — **only events/metadata** ("enclosure No. 7: anomalous activity", "queue A = 25 people"), raw video **does not leave** the node; upload on connectivity / via data-mule.

CV inference — on the edge; the cloud — training/retraining and aggregate analytics, not online inference. Models and rollout (OTA/versions/eval/drift) — **via AIP** (control plane, provider abstraction), uniformly across the whole park. Pipelines are enabled per node: an enclosure node — Animal Detection; a node at an entrance/near a ride — Crowd + Queue. Physically, platform instances are placed **at camera clusters** (PoE/network); this is one platform, not heterogeneous boxes.

**(B) Placement by the zone's task and biology (not "cameras/thermal everywhere"):**
- **Microclimate → MQTT sensors (Q4)**, not a thermal camera.
- **RGB — by default** (queues, crowd, behavior, feeding, re-ID, and detection of snakes/spiders) + **underwater RGB** for counting piranhas.
- **Thermal — pointwise, only where it is unique (thermal contrast exists):** fire/smoke; **overheating of ride mechanisms** (→ predictive maintenance, Q13); warm-blooded ones; **intruder/perimeter control at night**; an anonymous silhouette of people (searching for a child in the dark).
- **Detection/counting/escape of cold-blooded venomous ones → RGB-CV + enclosure breach sensors + RFID/PIT + weight/behavioral** (Q4/Q5), NOT thermal.

## Consequences
- + Works under poor connectivity, cheap on traffic, low latency of critical alerts, privacy by design; thermal is applied only where it physically works; climate — on cheap MQTT.
- − More expensive per camera (edge accelerators); harder to roll out models across the park; two camera types + breach/RFID complicate the park; thermal calibration.
- Mitigation: provider abstraction of models (AIP) + OTA updates + fitness functions; heavy models — offline in the cloud on samples; breach sensors are cheap and reliable for escape.

## TCO: edge vs cloud inference (order of magnitude)
- **Edge (our choice):** higher CapEx (~6 boxes × ~$800 ≈ $5k) + ownership (power/amortization/MLOps-OTA ≈ $220-500/month), but inference on-site, **video egress = 0**.
- **Cloud inference:** lower CapEx, but requires **streaming video outside**: ~19 cameras × even 2 Mbit/s (heavily compressed) ≈ 38 Mbit/s continuously → ~12 TB/month egress+ingest per camera park; under patchy Wi-Fi this is **physically unavailable**, and in money egress + cloud GPU inference easily exceeds the cost of edge ownership already at dozens of cameras.
- **Conclusion:** edge is cheaper by TCO **and** the only option under patchy Wi-Fi; cloud inference is rejected both on cost and on feasibility (§E).

## Considered / Rejected alternatives
- **Streaming all video to the cloud** — rejected: impossible under patchy Wi-Fi, expensive by egress, worst privacy.
- **Thermal for microclimate** — rejected: MQTT sensors measure climate directly, more accurately and many times cheaper.
- **Thermal for detection/health of venomous ones** — rejected: they are cold-blooded, thermal contrast ≈ 0 (a common naive mistake).
- **Thermal everywhere / RGB only** — rejected: the first — excessive capex; the second does not solve fire/overheating/night control/anonymity.

## Traceability
§E (patchy Wi-Fi, data delivery, MQTT budget) + §D (animals) + §C/§B (rides/safety) → **offline-tolerance/Cost/Privacy/Accuracy** → edge-CV (outside — only events) + climate on MQTT (Q4) + RGB (mass) + thermal (pointwise). Related ADRs: [ADR-013](ADR-013-privacy-surveillance-and-mobile-telemetry.md), [ADR-015](ADR-015-autonomous-shuttle-and-data-mule.md) (data mule for delay-tolerant metadata), [ADR-025](ADR-025-ride-attractions-management.md) (overheating of rides), AIP.
