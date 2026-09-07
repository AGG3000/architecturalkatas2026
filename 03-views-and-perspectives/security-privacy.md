# Security & Privacy perspective

## Security

### Baseline controls by domain
- **Ticketing (Q1):** PCI isolation, payments via an external PSP (we do not store cards, we store only the transaction token).
- **Identity (Q8):** external IdP, roles (visitor/keeper/admin), OIDC/OAuth2, short-lived tokens.
- **Edge devices:** authentication on upload (data mule), buffer encryption, tamper-resistance.

### Zero-trust between quanta
Principle: the network is untrusted, every call/event is authenticated and authorized independently — "never trust, always verify." (assumption about the implementation)
- **A service identity per quantum** — mTLS between quanta and to the bus; the publisher/subscriber of an event proves identity (SPIFFE/SVID class, workload identity). No "trusted perimeter."
- **Least-privilege authorization** — each quantum gets access only to its own topics/tables; events are signed, the consumer verifies the source and the schema (schema registry).
- **Segmentation** — the edge network (Q2/Q4/Q10/cameras) is separated from the cloud business quanta; only events/metadata go out from the edge (not raw data), traffic passes through a controlled ingest gateway.
- **Idempotency + event signing** — protection against replay under at-least-once delivery (event id key + signature/nonce).

### Secrets management
- **A centralized vault** (a managed secret store of the AWS Secrets Manager / HashiCorp Vault class) — secrets not in code and not in images. (assumption about the tool)
- **Short-lived dynamic credentials** — DBs/PSP/AI providers issue rotatable tokens; we avoid long-lived static keys.
- **AI providers behind the AIP** — LLM/CV/autonomy API keys live in the vault and are injected only into the AIP gateway, the business quanta do not see the keys.
- **Edge secrets** — device-key provisioning via device identity/certs; OTA rotation; on device compromise — certificate revocation, the buffer is encrypted (a hardware leak ≠ a data leak).

### Threat model for the sensitive
- **Payments (Q1):** threats — card theft, amount tampering, order replay. Controls — moving cards out to the PSP (out of the system scope), strict transaction consistency, idempotent order keys, audit log; PCI scope minimized to the token.
- **Edge devices (Q2 gate, Q4 IoT, cameras):** threats — physical access/theft, telemetry tampering/spoofing, injection of false events, buffer interception. Controls — device identity + event signing (we do not accept unsigned telemetry), buffer encryption at-rest, tamper-resistance, authentication on data-mule upload, dedup/idempotency against replay.
- **Autonomous shuttle (Q10):** safety-critical, the highest priority. Threats — interception of the teleoperation channel, sensor spoofing (GPS/LiDAR/camera), commands from a forged source, connectivity deadlock. Controls — **fail-safe to stop on uncertainty/loss of connectivity** (not "guessing"), an authenticated and encrypted teleoperation channel, on-board autonomy does not depend on the cloud (real-time control is not hung on patchy Wi-Fi), sensor redundancy as protection against single-source spoofing, a strict ODD, incident logging. Provider lock-in is acknowledged as a risk (changing vendor = changing the physical platform).

## Privacy
- **Video surveillance:** inference at the edge, **raw frames do not leave the device**; only events go to the bus. Where sufficient — a **thermal silhouette** instead of RGB (anonymous). Do not store faces.
- **Mobile telemetry:** a pseudonymous **ticket ID**, linkage to identity only in Q8 on opt-in consent; on completion of the visit — **crypto-shredding**.
- **AI content (Q11):** only content of animals/territory, without visitor faces.

### Personal data and the "right to be forgotten"
- **PII minimization:** the system operates on a pseudonymous ticket_id; the real identity (name/e-mail/payment profile) is isolated in Q8/Q1 and pulled in only on explicit opt-in.
- **Right to be forgotten (GDPR class):** implemented via **crypto-shredding** — deleting the ticket_id key makes the entire behavioral history unrecoverable, without expensive cascading deletion across quanta; the personal linkage in Q8 is deleted on request. (jurisdiction/GDPR applicability — assumption, the country is not set in the brief)
- **Consent:** location tracking / personal route quest — only on explicit opt-in at onboarding; without consent the ticket/card/events still work.
- **Medical data:** animals are not personal-data subjects; for visitors we collect no sensitive medical data by design (analytics — de-identified counters/heatmap, not face identification).
