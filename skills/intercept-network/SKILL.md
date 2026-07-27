---
name: intercept-network
description: Use this skill to control and observe backend communication in a test project — mock responses (force HTTP 500/401/empty/timeout/abort/edge states without the real backend) and capture/assert requests, responses and analytics events (Firebase, GA4, PostHog, Datadog, Customer.io). Triggers on "mock backend", "intercept requests", "capture traffic", "verify API call", "assert analytics", "simulate network error", "return empty response", "abort request", "proxy mobile traffic", "mockear respuestas", "capturar analytics", "interceptar la api". Works across web, mobile and API projects by detecting the safest available interception strategy.
version: 0.1.0
allowed-tools: Bash, Read, Write, Edit, Glob, Agent, AskUserQuestion
argument-hint: "[mock|capture] <description>  e.g. mock POST /payments returning 500"
---

# intercept-network

Control and observe backend communication for tests. Two capabilities:

- **mock** — intercept requests and return controlled responses to reproduce states that are hard or inconvenient against the real backend (errors, empty lists, timeouts, partial data, aborted requests).
- **capture** — record and validate requests, responses and analytics events to confirm the app sends the right information.

Applicable to web, mobile and API projects **as long as at least one viable network-control strategy exists**. Never assume a project is Playwright or Appium — detect first, then pick the safest strategy. `pw-framework` and `coign-appium-tests` are reference adapters, not the whole architecture.

`$ARGUMENTS` holds the mode + description, e.g. `mock POST /payments returning 500`, `capture analytics event payment_started`, `verify no duplicate request is sent`, `check whether this mobile build supports HTTPS interception`.

## Core rule: detect → verify → then promise

Never claim interception works because a proxy is set, a CA is installed, `relaxedSecurity` is on, or the build is debug. **Prove it empirically** by triggering a known request and confirming the traffic is captured/decrypted. Report honestly what is and isn't supported.

## Workflow

1. **Preflight** — detect the environment and resolve the capability matrix. See `references/preflight-check.md` and `references/capability-model.md`.
   - Platform, automation runner, language, app framework, HTTP client(s), config mechanisms, proxy support, backend redirection, certificate pinning, QA-build availability.
2. **Select strategy** — pick the first viable option in this order (safest/most stable first):
   1. `runner-native` (Playwright/Cypress built-in interception)
   2. `base-url-redirect` (point the app at a mock server via config/env)
   3. `dependency-injection` (swap the HTTP client in a QA/test build)
   4. `explicit-proxy` (app receives proxy host/port via config)
   5. `system-mitm-proxy` (device system proxy + trusted CA — gated by viability check)
   6. `provider-observability` (Firebase DebugView, PostHog/Datadog/GA4 debug logs) when traffic can't be intercepted directly
   7. `unsupported` — report the limitation and the build/env change needed
3. **Set up** mock and/or recorder via the matching recipe (`references/*-recipe.md`), implementing the shared contract in `references/interception-contract.md`.
4. **Assert** using the capture/analytics models. Wait for specific requests, never fixed sleeps.
5. **Run** the test.
6. **Report**: selected strategy, capability matrix, files created/modified, mocks/recorders implemented, results, limitations, required build/env changes, and the final viability status.
7. **Teardown**: stop the proxy/mock server, restore device proxy + certs, remove temporary CA/files. Confirm nothing was left changed.

## Modes

### mock
Simulate: HTTP 500/401/403/404/409/429, empty list, incomplete data, latency, timeout, aborted request, modified real response, a sequence of responses, replayed fixtures, forced edge case.

### capture
Verify: a request was sent (method/url/query/headers/body), a response was received, an unknown endpoint appeared, an analytics event fired, an action did NOT trigger a request, a duplicate request, an unexpected retry. Build endpoint/analytics documentation.

Mode comes from `$ARGUMENTS` or user intent.

## Security (always)

Follow `references/security-and-redaction.md`: redact tokens/cookies/PII, generate a temporary per-run CA, never trust arbitrary certificates, never modify release builds, restore device config on teardown, do not persist raw traffic, do not capture production traffic without authorization.

## Pipeline integration

- Consumes test cases authored by `gen-test-cases` (network scenarios phrased tool-agnostically).
- Invoked by `implement-test-cases` whenever a case mentions mock/error/empty/timeout/analytics/request-verification/no-request/duplicate/retry.
- Records findings into the Knowledge Base (`application-map/endpoints.md`, `application-map/analytics.md`) with knowledge states `expected | implemented | observed | verified`. See `references/knowledge-base-integration.md`.

## Reference files

- `capability-model.md` — capabilities, status enum, how to resolve the matrix.
- `preflight-check.md` — detection + viability report.
- `interception-contract.md` — shared `NetworkInterceptor` contract, mock/capture/assertion models, lifecycle, isolation.
- `analytics-capture.md` — provider filters, decoding, event semantics, dynamic-data handling.
- `playwright-recipe.md` — web (`runner-native`).
- `configurable-backend-recipe.md` — `base-url-redirect` + mock servers.
- `mobile-proxy-recipe.md` — mockttp/mitmproxy, Android + iOS, CA, teardown.
- `flutter-debug-build-recipe.md` — QA build that trusts a single QA CA / configurable base URL.
- `knowledge-base-integration.md` — endpoints + analytics catalogues.
- `security-and-redaction.md` — sensitive data + rules.
- `troubleshooting.md` — common failure modes.
