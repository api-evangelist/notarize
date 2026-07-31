---
name: Send an eSign transaction
description: Create a Proof (Notarize) eSign transaction, attach documents with signer designations, activate it, and collect completed documents.
api: https://dev.proof.com/reference
operations:
  - createtransaction
  - adddocument
  - updatedrafttransaction
  - activatedrafttransaction
  - gettransaction
---

# Send an eSign transaction

Collect legally binding electronic signatures without a notary meeting.

## Auth
- `ApiKey: <key>` or OAuth 2.0 Bearer token; base `https://api.proof.com`, `Content-Type: application/json`.

## Steps
1. **Create a draft** — `createtransaction` with `"draft": true` and the eSign transaction type, including signer(s) (`email` required).
2. **Add documents with designations** — `adddocument` and set signer designations / field instructions so signature fields are placed. eSign requires designations; creating with `"draft": false` before designations exist returns `422 "Esign required yet no Signer Designations specified in Document Bundle"`.
3. **Adjust if needed** — `updatedrafttransaction` to fix signers/documents before sending.
4. **Activate** — `activatedrafttransaction` to send the signing invitation (email, plus SMS if a phone is provided).
5. **Track & collect** — poll `gettransaction` for `detailed_status` (`esign_complete` / `complete`) or subscribe to `transaction.completed` and `transaction.released`; download documents after `transaction.released`.

## Rules
- eSign transactions may be tested in Production; fake-document notarizations may not (use Fairfax / IHN for notarizations).
- Same error envelope `{ "errors": [...] }` and 30 MB / 500 MB document limits apply.
