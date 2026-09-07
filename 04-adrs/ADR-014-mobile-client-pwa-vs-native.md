# ADR-014: Mobile Client — PWA vs Native (provider abstraction for geolocation)

## Status
Accepted

## Context
We need a consumer mobile client carrying 5 functions: route-quest, telemetry offload by ticket ID, operation on poor networks, feedback collection, event notifications. Case constraints: **limited budget** (recently loss-making estate — fact), **patchy Wi-Fi** (fact), the need for **offline operation** (caching tickets/map/route), geolocation for the route-quest and heatmap, push notifications. Candidates: PWA, native (iOS+Android), hybrid (Cordova/Capacitor). Affected: **Cost, offline-tolerance, Portability, time-to-market**.

## Decision
**PWA (installable, service worker) as the primary client** — a single web+mobile codebase, cheap, covering offline cache (service worker + IndexedDB), push (Web Push / FCM/APNs) and coarse geolocation. **Native is a second-iteration option** only when deep device access is needed (high-accuracy background geolocation, BLE beacons at enclosures/rides for precise positioning). Geolocation sits behind a **provider abstraction** (a "location provider" interface) so that GPS-PWA can be swapped for BLE-native without rewriting the route domain.

## Consequences
- + A single codebase and team, fast start, low cost, offline out of the box, no store-publishing overhead at launch.
- − Weaker background geolocation and BLE than native; iOS has historically restricted Web Push/background tasks; precise in-park positioning may require native/BLE later.
- Mitigation: provider abstraction for geolocation (easy to swap the provider); native is an evolutionary phase triggered by a signal of an accuracy need; **accessibility** — target **WCAG 2.2 AA**, an a11y gate in CI (axe-core / Lighthouse), **colorblind-safe** palettes for heatmap/route-quest, screen-reader support for key flows (entry / navigation / lost-child).

## Considered / Rejected alternatives
- **Native-first from the very start** — rejected: better UX/sensors, but expensive, two codebases, slow start; not justified for the budget and MVP.
- **Hybrid (Cordova/Capacitor)** — rejected: intermediate complexity and wrappers without a clear advantage over PWA for our functions.

## Traceability
Brief constraints (budget, patchy Wi-Fi, offline) → **Cost/offline-tolerance/Portability** → PWA + geolocation provider abstraction (Provider pattern). Related ADRs: [ADR-013](ADR-013-privacy-surveillance-and-mobile-telemetry.md), [ADR-015](ADR-015-autonomous-shuttle-and-data-mule.md) (BLE positioning from the shuttle).
