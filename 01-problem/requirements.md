# Requirements

## Functional
> Source: **FR1–FR7 — brief requirements §D**; **FR8 — derived from the §B asset** (collection of 40 attractions, not a separate §D requirement); **FR9–FR10 — design assumptions**.

| # | Requirement | Source | Domain |
|---|---|---|---|
| FR1 | Ticket purchase, including family passes, access to the grounds | §D | Q1 Ticketing |
| FR2 | Understand the popularity of different parts of the park | §D | Q3 Analytics |
| FR3 | Monitoring of animal health | §D | Q5 Animal Health |
| FR4 | Track how much/how well the animals eat | §D | Q5 Animal Health |
| FR5 | Control of the piranha population size | §D | Q5 Animal Health |
| FR6 | Grow the number of visitors | §D | Q6 Growth |
| FR7 | Increase profitability, attract returning guests | §D | Q6 Growth / Q11 |
| FR8 | Attractions management: availability, capacity, safety inspections, maintenance (40 units) | derived §B | Q13 Ride & Attractions Mgmt |
| FR9 | Visitor profile + preferences | assumption | Q8 + Q6 |
| FR10 | RFID token (wristband/pass): offline entry, cashless, location, family linkage | assumption | ADR-027 → Q1/Q2/Q3/Q8 |

## Non-functional (NFR with metrics)
Target numbers are estimates pending calibration; marked "(assumption)" where not a brief fact. The **"Verification"** column links each NFR to [fitness functions / V&V](../05-ai/validation-verification.md) — a metric without a way to verify it does not count.

| NFR | Target | Source | Verification |
|---|---|---|---|
| Scale | 5000 → 15000 visits/day within 3 years | brief fact | load testing, autoscale metrics |
| Availability | 3–4 nines (realistic for the budget) | (assumption) | SLO monitoring, error budget |
| Ticketing response | p95 ≤ 500 ms per purchase/validation | (assumption) | APM p95, fitness threshold |
| Heatmap/NL-analytics response | p95 ≤ 2 s | (assumption) | latency fitness function (V&V) |
| Animal telemetry freshness | real-time for critical, ≤ 5 min for routine | (assumption) | data-lag monitoring |
| **Latency "alert → keeper"** | critical welfare alert delivered ≤ 30 s | (assumption) | recall+latency fitness Q5 (V&V) |
| **AI accuracy (welfare)** | F1 ≥ 0.90; recall of critical conditions ≥ 0.95 | (assumption) | offline eval on golden-set (V&V) |
| **Confidence calibration** | uncertain-but-wrong within normal bounds | (assumption) | miscalibration fitness, PSI drift |
| **Inference cost** | cost-per-inference within the AIP budget | (assumption) | AI Gateway cost-monitor |
| **Privacy / retention** | PII pseudonymous; crypto-shredding; welfare logs 2–3 yrs | ADR-013 / governance | access audit, [security-privacy](../03-views-and-perspectives/security-privacy.md) |
| **Accessibility (guest PWA)** | WCAG 2.2 AA on key flows; colorblind-safe visualizations | (assumption) | a11y fitness in CI (axe-core/Lighthouse) + manual screen-reader audit ([validation-verification](../05-ai/validation-verification.md)) |
| Edge offline-tolerance | operation on Wi-Fi loss (store-and-forward) | from constraint E | degraded-mode test, [reliability](../03-views-and-perspectives/reliability.md) |

## Traceability to the brief's goals
Growth/profitability → FR6/FR7 (Q6/Q11) · "healthy animals" → FR3–FR5 (Q5) · "where to invest/staff" → FR2 (Q3). The full chain goal→characteristic→style→ADR — in `../04-adrs/`.
