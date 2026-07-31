---
name: Subscribe to Proof transaction webhooks (V2)
description: Discover available Proof (Notarize) webhook events, create a Webhooks V2 subscription, and verify inbound event signatures.
api: https://dev.proof.com/reference
operations:
  - getwebhooksubscriptionsv2
  - createwebhookv2
  - getallwebhooksv2
  - getwebhookeventsv2
  - createtestwebhook
---

# Subscribe to Proof transaction webhooks (V2)

Get notified of transaction/notary state changes instead of polling.

## Auth
- `ApiKey: <key>` or OAuth 2.0 Bearer token; base `https://api.proof.com`.

## Steps
1. **List subscribable events** — `getwebhooksubscriptionsv2` to see every event you can subscribe to (families `transaction.*` and `notary.*`).
2. **Create a subscription** — `createwebhookv2` with your endpoint URL and the events (use a wildcard like `transaction.*`, or specific events such as `transaction.completed`, `transaction.released`, `transaction.signer.kba_failed`). For OAuth-created subscriptions, include a `signing_key` to enable HMAC signing.
3. **Confirm** — `getallwebhooksv2` to list subscriptions; `createtestwebhook` to send a test event; `getwebhookeventsv2` to review delivery history.
4. **Verify each delivery** — recompute an HMAC-SHA256 (hex) of the raw request body with your signing key and compare to the `X-Notarize-Signature` header.

## Rules
- Respond `200` within 30 seconds. Non-200/timeout triggers back-off retries: up to 16 attempts over 48 hours.
- Deliveries are at-least-once and unordered — implement the receiver idempotently and skip to the latest `detailed_status`.
- Allow-list Proof outbound IP ranges: `44.204.166.96/28`, `35.89.72.128/28`, `3.145.244.0/28`.
- Listen for `transaction.released` (not just `transaction.completed`) before downloading documents in signer-paid workflows.
