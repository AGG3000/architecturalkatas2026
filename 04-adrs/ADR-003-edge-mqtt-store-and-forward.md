# ADR-003: Edge + MQTT store-and-forward instead of direct uplink to the cloud

## Status
Accepted

## Context
Wi-Fi across the park is **patchy** (brief fact, §E), but animal telemetry (health, how much and how they eat, control of the piranha population — brief fact, §D3) must be reliably delivered to the cloud (brief fact: "a way is needed to deliver data from the estate to the cloud", §E). There is a budget for **MQTT-compatible devices** to install throughout the park (brief fact, §E). Direct real-time uplink of every sensor to the cloud is impossible: on a connection loss readings are lost, and a surge of reconnections creates load (assumption).

## Decision
Telemetry ingestion — an **edge quantum with MQTT ingestion and a local store-and-forward buffer**. Sensors publish over MQTT to an edge broker/buffer near the enclosures; the buffer accumulates data during an outage and **syncs to the cloud in batches once connectivity returns**, idempotently by event id. Similarly, Gate scanners keep a local cache of valid tickets and operate offline. Rationale: the primary characteristic of these quanta is offline-tolerance, not immediacy.

## Consequences
- Pros: no data loss during outages, resilience to patchy Wi-Fi, smoothing of surges, cheap traffic (batches), direct use of the MQTT budget.
- Cons: latency in delivering routine telemetry (minutes), edge complexity (buffer, dedup, monitoring of "gaps" in data), on-site hardware required.
- Mitigations: classification of data as real-time vs delay-tolerant (see [ADR-004](ADR-004-edge-connectivity-mesh-cellular-data-mule.md)); at-least-once + idempotency by event id (see [ADR-007](ADR-007-event-reliability-inbox-outbox.md)); ping/health-check to detect breaks and "stale data" (assumption).

## Considered / Rejected alternatives
- **Direct real-time uplink of every sensor to the cloud** — rejected: under patchy Wi-Fi it loses readings on outages, is expensive in reconnections and traffic, and makes monitoring availability dependent on the worst coverage point.
- **Store everything only on the edge, sync manually** — rejected: no timely analytics and alerts, manual collection does not scale to 55 enclosures, the value of the cloud is lost.

## Traceability
Brief goal "animal monitoring + data delivery to the cloud under patchy Wi-Fi, budget for MQTT" → characteristic offline-tolerance·Reliability·Throughput → edge + MQTT store-and-forward. Related ADRs: [ADR-001](ADR-001-event-driven-architecture-and-quanta.md), [ADR-004](ADR-004-edge-connectivity-mesh-cellular-data-mule.md), [ADR-007](ADR-007-event-reliability-inbox-outbox.md).

**Mitigates risks:** R3 (see [risk-register](../03-views-and-perspectives/risk-register.md)).
