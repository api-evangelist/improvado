---
name: Register and verify a webhook endpoint
description: Subscribe to Improvado load/transformation/extraction events, verify the endpoint via the challenge handshake, and validate delivery signatures.
api: Improvado Embedded API v3
base_url: https://embedded.improvado.io
operations:
  - POST /api/v3/token
  - GET /api/v3/webhook_event_types/
  - POST /api/v3/webhook_endpoints/
  - POST /api/v3/webhook_endpoints/{id}/verify/
  - POST /api/v3/webhook_endpoints/{id}/regenerate_secret/
---

# Register and verify a webhook endpoint

Receive Improvado pipeline events on your own HTTPS receiver.

## Auth
1. Mint a workspace-scoped Bearer token: `POST /api/v3/token` (Basic auth first). Send it as
   `Authorization: Bearer <token>` on all calls below.

## Steps
1. **Discover event types** — `GET /api/v3/webhook_event_types/`. Never hardcode; subscribe to the
   returned identifiers (e.g. `load_started`, `load_completed`, `transformation_started`,
   `transformation_completed`, `extraction_created`, `extraction_paused`).
2. **Create the endpoint** — `POST /api/v3/webhook_endpoints/` with your HTTPS URL and the event
   types. You receive a signing secret.
3. **Verify** — Improvado sends a `GET` to your URL carrying `X-Improvado-Verify-Token` and
   `X-Improvado-Challenge`. Respond `200` and echo the challenge value as the body to activate.
   You can also trigger `POST /api/v3/webhook_endpoints/{id}/verify/`.

## Receiving deliveries
- Every POST delivery includes `X-Improvado-Signature` — an HMAC-SHA256 of the body computed with
  your signing secret. Recompute and compare before trusting the payload.
- De-duplicate on the `X-Improvado-Idempotency-Key` header (deliveries are retried up to 4 times
  with backoff `[0s, 2s, 5s, 10s]`).
- Respond `2xx` within seconds to acknowledge, or the delivery is retried.

## Maintenance
- Rotate the signing secret with `POST /api/v3/webhook_endpoints/{id}/regenerate_secret/` and
  update your verifier.
