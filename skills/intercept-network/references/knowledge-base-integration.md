# Knowledge Base integration

Interception discovers real endpoints and analytics events. Persist them in the project's `application-map/` so coverage becomes reusable knowledge, shared with the other skills.

## Files

```
application-map/endpoints.md
application-map/analytics.md
```

## Endpoint format

```md
## POST /payments/:paymentId/confirm
- Method: POST
- Environment: staging
- Authentication: bearer token
- Request fields: paymentId, confirmationType
- Observed responses: 200, 409, 500
- Last observed build: 2.4.0-qa
- Status: observed
```

## Analytics format

```md
## payment_started
- Provider: PostHog
- Trigger: User presses the Pay button
- Observed properties: payment_method, currency, amount
- Dynamic properties: timestamp, session_id
- Platforms: Web, Android
- Status: observed
```

## Knowledge states

Each endpoint/event carries one of:

```
expected      part of the intended spec (from a requirement or test case)
implemented   an automated test asserts it
observed      seen on the wire during a run
verified      observed AND matches the expected spec
```

An **observed** item never auto-promotes to **expected**. Seeing traffic is not the same as it being correct behavior — a human (or a test case) decides what is expected. This keeps the knowledge base from turning accidental or buggy traffic into "the spec".

## Rules

- Apply redaction (see `security-and-redaction.md`) before writing anything — no tokens, cookies or PII in these files.
- Update controllably: add newly observed items, update statuses, never overwrite human-authored `expected` notes.
- Prefer diffs a human can review over bulk rewrites.
