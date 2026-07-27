# Security & redaction

Interception touches real traffic, so treat everything as sensitive by default.

## Redact these headers

```
authorization
proxy-authorization
cookie
set-cookie
x-api-key
x-auth-token
```

## Redact these body fields

```
password
token
access_token
refresh_token
email
phone
card_number
cvv
device_id
session_id
```

Redact before logging, before writing to the Knowledge Base, and before attaching anything to a report. Replace values with `***` rather than dropping keys, so structure stays readable.

## Rules

- Do not automatically persist raw traffic. Capture in memory for assertions; write only redacted summaries if needed.
- Do not commit private keys or CA private material to the repo. Generate a **temporary CA per workspace/run** and delete it on teardown.
- Never trust arbitrary certificates. Trust exactly one QA CA when a build change is required.
- Never apply QA network config to release builds; the release must fail if it is still enabled.
- Restore device proxy and certificate configuration on teardown.
- Do not capture production traffic without explicit authorization. Default to non-prod environments.
- Do not record unnecessary personal information.
