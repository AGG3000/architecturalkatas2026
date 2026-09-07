# ADR-027: Visitor RFID token (wristband/pass) in addition to the app

## Status
Accepted *(assumption: not directly specified in the brief; park-standard, Disney MagicBand class)*

## Context
The mobile app (PWA, [ADR-014](ADR-014-mobile-client-pwa-vs-native.md)) is a rich UI, but: some visitors are **without a smartphone**, the battery dies, and **Wi-Fi is patchy** (§E) → a pure-app approach is fragile at entry/payment. We need a physical token that works offline and without a phone. We also need family passes (§D) and a link with lost-child search (the safety channel already exists).

## Decision
Introduce a **visitor RFID token** (wristband/pass card) as a **client token layer alongside the app** (not a new quantum). RFID-reader taps integrate into the existing quanta:
- **Q2 Access & Gate** — entry/access to rides, **offline validation** (a cache of valid tokens on the reader under patchy Wi-Fi);
- **Q1 Ticketing** — **cashless purchases** by token (settlement via the PSP, tied to a balance/prepayment);
- **Q3 Analytics** — **passive location** via RFID readers → complements/cheapens the heatmap vs GPS-app;
- **Q8 Identity** — tying the token to a profile and a **family link** (parent↔child) → strengthens lost-child.
The app remains the primary UI; RFID is an offline physical token for those without the app. Data — a pseudonymous `token_id` (like ticket_id, privacy [ADR-013](ADR-013-privacy-surveillance-and-mobile-telemetry.md)).

## Consequences
- Pros: works without a smartphone/during an outage/on a dead battery; cheap location (heatmap); cashless speeds up purchases; family link and lost-child; covers visitors outside the app.
- Cons: capex on wristbands/readers + the logistics of issuance/return/sanitizing; another identity channel (consistency with the app profile is needed); location privacy.
- Mitigation: a pseudonymous token_id + opt-in for location + crypto-shredding ([ADR-013](ADR-013-privacy-surveillance-and-mobile-telemetry.md)); readers at entries/rides/points of sale (not "everywhere"); RFID and the app are one profile (Q8).

## Considered / Rejected alternatives
- **The app only (no RFID)** — rejected: fragile under patchy Wi-Fi/without a smartphone/on a dead battery; worse for families/children and offline payment.
- **RFID instead of the app** — rejected: we lose the rich UI (route-quest, chat, profile); RFID is an addition, not a replacement.
- **BLE beacons in the phone instead of RFID** — rejected as the sole option: does not work without a phone/app; RFID is cheaper as a mass passive token (but BLE is possible as an option for app owners).

## Traceability
§E (patchy Wi-Fi) + §D (tickets/family passes) + safety (lost-child) → **Availability/offline-tolerance/Usability** → an RFID token as an offline complement to the app. Related ADRs: [ADR-002](ADR-002-quantum-boundaries-by-characteristics.md) (Q1/Q2/Q3/Q8), [ADR-013](ADR-013-privacy-surveillance-and-mobile-telemetry.md) (privacy of token_id), [ADR-014](ADR-014-mobile-client-pwa-vs-native.md) (app), [ADR-006](ADR-006-pci-ticketing-isolation-external-psp.md) (cashless via the PSP).
**Mitigates risks:** R14 (privacy — pseudonym/crypto-shredding) (see [risk-register](../03-views-and-perspectives/risk-register.md)).
