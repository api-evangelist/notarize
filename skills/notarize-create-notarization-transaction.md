---
name: Create and complete a notarization transaction
description: Create a Proof (Notarize) remote online notarization transaction, attach documents, send it to a signer, and retrieve the notarization record once complete.
api: https://dev.proof.com/reference
operations:
  - createtransaction
  - adddocument
  - activatedrafttransaction
  - gettransaction
  - getnotarizationrecord
---

# Create and complete a notarization transaction

Use the Proof API (formerly Notarize) to run a Remote Online Notarization (RON) end to end.

## Auth
- Send `ApiKey: <key>` (Full Access key, prefix `prf_`) OR an OAuth 2.0 Bearer token from `POST https://api.proof.com/oauth/v2/token` (client_credentials).
- Production base `https://api.proof.com`; Fairfax sandbox base `https://api.fairfax.proof.com`.
- Always set `Content-Type: application/json`.

## Steps
1. **Create a draft transaction** — `createtransaction` with `"draft": true`, the signer(s) (`email` is the only required signer field), and transaction options. Capture the returned transaction id (`ot_...`).
2. **Attach documents** — `adddocument` for each PDF. Wait for the `transaction.document.processed` webhook (white-text tags / template matching finished) before activating.
3. **Activate** — `activatedrafttransaction` to send the transaction to the signer. Do NOT create with `"draft": false` when documents need eSign designations — that returns `422 "Esign required yet no Signer Designations..."`; the draft-then-activate flow avoids it.
4. **Track status** — poll `gettransaction` and read `detailed_status` (draft → sent_to_signer → meeting_in_progress → complete), or subscribe to webhooks (see the webhook skill).
5. **Retrieve the record** — on `transaction.completed` / `transaction.released`, call `getnotarizationrecord` to fetch the meeting video and audit evidence.

## Rules
- Individual document max 30 MB; total bundle max 500 MB (else `422`).
- Errors come back as `{ "errors": ["message"] }`. Log the `X-Amz-Trace-Id` response header for Proof support.
- Test in Fairfax with an In-House Notary profile or by mock-firing the `transaction.completed` webhook; real verifications are not performed in Fairfax.
