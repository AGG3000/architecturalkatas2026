# Glossary (abbreviations and terms)

| Term | Meaning |
|---|---|
| **AIP / AI Gateway** | AI Platform — an end-to-end gateway to all AI models: provider abstraction, Model Router, Prompt Adapter, Evaluation Engine, guardrails, cost-monitor, fallback |
| **Quantum (architectural quantum)** | an independently deployable component with high cohesion and its own DB; boundaries drawn along diverging quality characteristics |
| **CQRS** | Command Query Responsibility Segregation — separation of reads (analytics) and writes (operations) |
| **CDC** | Change Data Capture — capturing changes from operational DBs into the lakehouse without breaking quantum autonomy |
| **Medallion (bronze/silver/gold)** | lakehouse layers: raw → cleaned → business metrics |
| **Feature Store** | a unified store of online/offline features for ML models |
| **Semantic Layer** | defined business metrics on top of gold — for consistent NL analytics |
| **Digital Twin** | "digital twin" = up-to-date domain projections/aggregates on top of the Lakehouse (not a separate component) |
| **RAG** | Retrieval-Augmented Generation — LLM answers based on knowledge-base search with source citations |
| **DTN / data-mule** | Delay-Tolerant Networking; the shuttle physically carries buffered data from zones without connectivity to the cloud |
| **Edge AI Platform** | a unified edge runtime on cameras: Camera Processing · Animal Detection · Crowd Analytics · Queue Monitoring · Local Buffering |
| **Store-and-forward** | buffering data at the edge when connectivity drops, with later forwarding |
| **HITL** | Human-in-the-loop — a human in the decision loop for critical AI |
| **V&V** | Validation & Verification — how we make sure the (non-deterministic) AI works |
| **Fitness function** | an automated, measurable check of a characteristic/quality (including AI) against a threshold |
| **ODD** | Operational Design Domain — the defined operating conditions for autonomous driving |
| **RAID / Risk register** | a consolidated list of risks (Risk → likelihood/impact → mitigation → owner) |
| **PSP** | Payment Service Provider — external payment provider (we do not store cards) |
| **PCI** | payment card security standard; scope minimized to a token |
| **IdP** | Identity Provider — external authentication provider |
| **PWA** | Progressive Web App — an installable web application with an offline cache |
| **MQTT** | a lightweight messaging protocol for IoT devices |
| **BFF** | Backend-for-Frontend — a backend tailored to a specific client |
| **RFID / PIT** | radio-frequency tag (visitor wristband/pass; animal chip tag) |
| **PSI** | Population Stability Index — how far the data distribution has "drifted" from the training one (a model-drift signal) |
| **Recall** | the share of real cases the model **caught** (a miss is costly → critical for welfare/safety) |
| **Precision** | the share of triggers that turned out **correct** (higher → fewer false alarms) |
| **F1** | a balanced detection score (combines recall and precision into a single number 0–1; higher is better) |
| **MAE / MAPE** | mean forecast/count error (MAE — in units, MAPE — in %): e.g. how far the AI is off in counting the piranha population or queue length |
| **SoT** | Source of Truth (operations — in quanta; analytics/ML — in the lakehouse) |
| **SLA / RPO / RTO** | target service level / acceptable data loss / recovery time |
| **ticket_id / token_id** | pseudonymous visitor identifier (privacy, crypto-shredding) |

*Quanta Q1–Q13 ↔ business capabilities — see the table in the [README](README.md#quanta--business-capabilities).*
