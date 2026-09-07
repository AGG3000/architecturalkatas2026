# Constraints and Assumptions

## Constraints (brief facts, section E)
- **Patchy Wi-Fi** across the grounds → edge inference, store-and-forward, data mule.
- Cloud is allowed, but **a channel to deliver data from the estate to the cloud is needed**.
- There is a **budget for MQTT-compatible devices** to install across the park.

## Assumptions (NOT in the brief — marked explicitly)
| Assumption | Value | Where used |
|---|---|---|
| Estate area | ≈100 acres operational (compact ≈60 / sprawling ≈150+) | connectivity threshold estimate |
| Number of remote enclosures | ≈5–40 (out of 55) depending on area | break-even gateways vs data mule |
| Ticket pricing | not specified | cost/ROI estimates |
| Legacy architecture | absent (we design greenfield) | "alignment of AI characteristics with the existing architecture" we interpret as consistency with our own baseline |
| Animal species | none specified other than piranhas | generalized welfare monitoring |
| Number of shuttles | MVP ≈3, at scale ≈5–6 | fleet for passenger demand + data mule |
| Monetization of carnivorous plants | not in §D; we treat it as a revenue opportunity | feeding show/content/quest zone (quantum reuse) |

*Rule: any number/feature not in `brief-facts.md` is marked "(assumption)".*
