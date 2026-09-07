# Deployment view (edge / cloud / network)

The deployment diagram (edge/channel/cloud tiers) is in [../diagrams/deployment.md](../diagrams/deployment.md).

## Tiers
- **Edge (on the estate, patchy Wi-Fi):** gate controllers (offline ticket cache), MQTT broker + edge buffer (store-and-forward), Fixed Cameras (edge-CV inference), the shuttle as a mobile edge node.
- **Estate→cloud channel:** hybrid — fixed Wi-Fi where cost-effective · Wi-Fi mesh (shuttle) · data mule (remote enclosures) · cellular fallback for the real-time-critical. The selection threshold gateways vs data mule: N*≈2–6 remote enclosures; the more sprawling the estate, the more decisively the data mule wins.
- **Cloud:** business quanta (event backbone, DBs, analytics warehouse), AIP.

## Data classification by latency
- **Real-time-critical** (venomous-animal alert, payment, missing-child search) → fixed/cellular uplink (backup — dual-carrier/satellite; behavior on channel/carrier failure and the degradation matrix are in [reliability.md](reliability.md), [ADR-017](../04-adrs/ADR-017-connectivity-failure-degraded-operation.md)).
- **Delay-tolerant** (routine telemetry, metrics, 360 recording) → data mule/mesh.

## Principles
Only events/metadata leave the edge — **not raw video/stream**. Region: at MVP — a single region with multi-AZ (protection against a zone failure at a reasonable cost, low latency to the estate); multi-region — roadmap as Availability requirements grow (DR/geo-redundancy), since at the start this is unjustified overhead.
