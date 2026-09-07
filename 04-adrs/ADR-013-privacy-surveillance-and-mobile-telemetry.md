# ADR-013: Privacy — surveillance (edge, thermal anonymity) + mobile telemetry (pseudonymous ticket ID + crypto-shredding)

## Status
Accepted

## Context
Two streams of personal data: (1) **surveillance** by dozens of cameras across the park and (2) **mobile telemetry** — visitors' location/actions/purchases (Q3). Visitors — including **children** (family passes, brief fact). Privacy compliance is required (GDPR class — assumption about jurisdiction) without losing analytical value. Affected: **Security/Privacy, Analyzability**.

## Decision
**Video:** inference on the edge, **raw frames do not leave the device** (see ADR-011); to the bus — only events/counters; where anonymity is sufficient — a **thermal silhouette instead of RGB** (does not identify a face); faces are not stored, for analytics — anonymized heatmaps/counters.
**Mobile telemetry:** travels under a **pseudonymous ticket ID**, not under an identity; the "ticket ID → person" link lives only in Q8 (Identity) and is revealed only on **opt-in consent** to tracking. The "right to be forgotten" upon completion of the visit — **crypto-shredding**: deleting the ticket ID key makes the movement history unrecoverable.

## Consequences
- + Privacy by design; PII minimization; fulfillment of the "right to be forgotten"; analytics on anonymized data is preserved; legal defensibility.
- − Without consent, the personalized route-quest is unavailable (UX degradation); crypto-shredding requires discipline in key management; pseudonymization complicates debugging/session linking.
- Mitigation: a consent screen in onboarding (ticket/card/events work without it too); rotation and a secure key store; audit of access to the link in Q8.

## Considered / Rejected alternatives
- **Store raw video and PII centrally for "rich" analytics** — rejected: egress under patchy Wi-Fi, privacy and legal risk, more expensive.
- **Full anonymization without a ticket ID** — rejected: breaks the personalized route-quest, feedback, and return rate (the brief's business goals).

## Traceability
Brief goal (zone popularity analytics, family passes/children, return rate) → **Privacy/Security + Analyzability** → edge video + thermal anonymity + pseudonym + crypto-shredding. Related ADRs: [ADR-011](ADR-011-fixed-cameras-edge-cv-vs-cloud-cv.md), [ADR-014](ADR-014-mobile-client-pwa-vs-native.md).

**Mitigates risks:** R14 (see [risk-register](../03-views-and-perspectives/risk-register.md)).
