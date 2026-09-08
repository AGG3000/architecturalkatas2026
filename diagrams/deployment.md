# Deployment — edge tiers → communication channels → cloud

**What it shows:** the physical placement and connectivity under patchy Wi-Fi — three edge tiers → heterogeneous channels → cloud + AIP.

Physical placement and **connectivity under patchy Wi-Fi**. Three tiers: edge devices in the estate → heterogeneous channels (Wi-Fi / mesh / data-mule / cellular) → cloud + AIP. Data is classified by delay tolerance: **real-time-critical** goes over a fixed/cellular channel, **delay-tolerant** — via the shuttle data mule.

> **Phases:** the diagram is the target architecture. In the **MVP** the channels = fixed Wi-Fi/mesh + **cellular uplink** (dual-carrier/satellite) + edge store-and-forward; **manned electric transport + data-mule — MVP** (they reduce the cost of connecting remote enclosures, not an MVP condition); shuttle **autonomy** — R3. See [mvp-vs-roadmap](../02-solution/mvp-vs-roadmap.md).

```mermaid
flowchart TB
    %% ==== EDGE TIER ====
    subgraph EDGE["EDGE TIER — estate (patchy Wi-Fi)"]
        direction LR
        gate["Gate controllers (Q2)<br/>offline-cache of tickets"]:::edge
        iot["MQTT enclosure sensors (Q4)<br/>store &amp; forward buffer"]:::edge
        cams["Edge AI Platform (RGB/Thermal cameras)<br/>Camera Proc · Animal Detection ·<br/>Crowd/Queue · Local Buffering"]:::edge
        shuttle["Shuttle (Q10) — mobile edge node<br/>on-board autonomy · buffer · WiFi-mesh AP · 360 · display"]:::mobile
    end

    %% ==== COMMUNICATION CHANNELS ====
    subgraph LINKS["COMMUNICATION CHANNELS (chosen by data class and zone)"]
        direction LR
        wifi(["Fixed Wi-Fi<br/>(where cost-effective)"]):::link
        mesh(["WiFi-mesh<br/>(mobile AP along the route)"]):::link
        mule(["Data mule / DTN<br/>(manned transport carries the buffer, MVP; autonomy R3)"]):::link
        cell(["Cellular uplink<br/>(fallback, real-time only)"]):::link
        bridge(["PtP/PtMP radio bridge<br/>(LoS: real-time for Edge/remote)"]):::link
    end

    %% ==== CLOUD ====
    subgraph CLOUD["CLOUD TIER"]
        direction LR
        ingest["Cloud ingest<br/>(idempotent upload)"]:::cloud
        backbone[["Event Backbone<br/>(topics / streams)"]]:::bus
        quanta["Business quanta Q1·Q3·Q5·Q6·Q7·Q8·Q9·Q11·Q12<br/>(containers / serverless scale-to-zero)"]:::cloud
        stores[("DBs: operational · analytics warehouse ·<br/>time-series · object storage/CDN")]:::db
        aip["AIP — AI Platform<br/>provider abstraction · guardrails · eval · cost-monitor"]:::aip
    end
    providers["LLM / CV / autonomy providers"]:::ext

    %% ==== EDGE → CHANNELS ====
    gate -->|"GateScanned (real-time)"| wifi
    gate -.->|"fallback"| cell
    cams -->|"critical alerts<br/>(escape/fire/fall)"| wifi
    cams -.->|"fallback"| cell
    cams -. "delay-tolerant metadata" .-> mule
    iot -. "routine telemetry (drive-by)" .-> mule
    iot -->|"if coverage exists"| wifi
    cams -->|"real-time over LoS"| bridge
    iot -->|"real-time over LoS"| bridge
    shuttle --- mesh
    shuttle -. "carries the enclosure/camera buffer" .-> mule
    shuttle -->|"upload at the stop"| wifi
    shuttle -.->|"real-time alert"| cell

    %% ==== CHANNELS → CLOUD ====
    wifi --> ingest
    mesh --> ingest
    mule -->|"batch upload (idempotent)"| ingest
    cell --> ingest
    bridge --> ingest

    %% ==== CLOUD internal ====
    ingest --> backbone
    backbone <--> quanta
    quanta --- stores
    quanta -.->|"inference"| aip
    aip --> providers

    %% ==== LEGEND (in the diagram so it appears in the SVG) ====
    subgraph LEGEND["Legend"]
        direction LR
        lg_edge["Edge device"]:::edge
        lg_mobile["Mobile edge (shuttle)"]:::mobile
        lg_link(["Communication channel"]):::link
        lg_cloud["Cloud component"]:::cloud
        lg_bus[["Event backbone"]]:::bus
        lg_db[("Storage")]:::db
        lg_aip["AIP"]:::aip
        lg_ext["External system"]:::ext
    end

    classDef edge fill:#0f9d58,stroke:#0a6b3c,color:#fff;
    classDef mobile fill:#0b8043,stroke:#064d28,color:#fff;
    classDef link fill:#f4b400,stroke:#b8860b,color:#000;
    classDef cloud fill:#1168bd,stroke:#0b4884,color:#fff;
    classDef bus fill:#f4b400,stroke:#b8860b,color:#000;
    classDef db fill:#5b6770,stroke:#3a4148,color:#fff;
    classDef aip fill:#a64d79,stroke:#741b47,color:#fff;
    classDef ext fill:#999999,stroke:#6b6b6b,color:#fff;
```

## Legend

| Shape / color | Meaning |
|---|---|
| Green rectangle | Edge device (stationary) in the estate |
| Dark-green rectangle | Mobile edge node (shuttle Q10) |
| Yellow oval | Communication channel |
| Blue rectangle | Cloud component (quanta, ingest) |
| Yellow block (double border) | Event backbone |
| Gray cylinder | Data storage |
| Purple rectangle | AIP |
| Solid arrow | Real-time / main flow |
| Dashed arrow | Delay-tolerant (data mule) or fallback |

## Description of tiers and channels

**EDGE TIER (estate, patchy Wi-Fi).** Gate controllers (Q2) with an offline ticket cache; MQTT enclosure sensors (Q4) with a store-and-forward buffer; fixed cameras with edge-CV compute boxes; the shuttle (Q10) as a mobile edge node (on-board autonomy, buffer, WiFi-mesh AP, 360 cameras, info-display). Only events/metadata leave outward — raw video and telemetry stay on the edge.

**Communication channels (hybrid, chosen by data class and zone):**
- **Fixed Wi-Fi** — where cost-effective; the primary channel for real-time.
- **WiFi-mesh** — a mobile AP on the shuttle patches coverage along the route.
- **Data mule / DTN** — the shuttle physically carries buffered delay-tolerant data from remote enclosures and uploads idempotently at a stop with connectivity (cheaper than a gateway at each of the 55 enclosures).
- **Cellular uplink** — fallback for real-time-critical events ("animal escaped," fire, purchase, child search) where there is no LoS/bridge.
- **Directional radio bridges (PtP/PtMP)** — a live channel for stationary Edge AI nodes and remote zones **with line-of-sight**; cheaper than cellular in operation, available already in the MVP. LoS/vegetation/weather/alignment risks and mitigations (site survey, backup/mesh, fiber for critical zones) — see [ADR-004](../04-adrs/ADR-004-edge-connectivity-mesh-cellular-data-mule.md), risk R21.

**Data classification:** real-time-critical → Wi-Fi/cellular (seconds); delay-tolerant (routine telemetry, drive-by snapshots, queue metrics) → data mule (minutes to tens of minutes).

**CLOUD TIER.** Cloud ingest (idempotent upload, at-least-once + dedup by event id) → event backbone → business quanta (containers/serverless scale-to-zero) with separated DBs (operational, analytics warehouse, time-series, object storage/CDN). The AIP serves inference for all AI quanta and reaches out to external LLM/CV/autonomy providers through the provider abstraction.
