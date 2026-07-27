# Capability model

The skill first resolves what it can actually do in the current project, then chooses a strategy. Never promise a capability that hasn't been verified.

## Capabilities

```ts
type NetworkCapabilities = {
  captureRequests: Status
  captureResponses: Status
  inspectRequestBody: Status
  inspectResponseBody: Status
  mockResponses: Status
  modifyRequests: Status
  modifyResponses: Status
  abortRequests: Status
  injectLatency: Status
  simulateTimeout: Status
  inspectWebSockets: Status
  decryptHttps: Status
  captureAnalytics: Status
}
```

## Status values

```
SUPPORTED                 works now, verified empirically
PARTIALLY_SUPPORTED       works for some traffic only (e.g. webview yes, native no)
BUILD_CHANGE_REQUIRED     needs a QA build change (CA trust / DI / base-url)
ENVIRONMENT_CHANGE_REQUIRED  needs device/emulator/env change (proxy, cert store)
UNSUPPORTED               not achievable with any available strategy
INCONCLUSIVE              could not be determined; needs a re-run or more info
```

## Resolving the matrix

1. Run the preflight (see `preflight-check.md`) to detect stack, clients, proxy/cert support.
2. For each capability, start at `INCONCLUSIVE`.
3. Promote to `SUPPORTED` only after the empirical viability check observes it working.
4. If the check shows a `CONNECT` tunnel but no decryption → `BUILD_CHANGE_REQUIRED`.
5. If the app ignores the proxy entirely → `ENVIRONMENT_CHANGE_REQUIRED` or `INCONCLUSIVE`.
6. If only some SDKs/domains are visible → `PARTIALLY_SUPPORTED`.

## Example report

```yaml
strategy: runner-native
capabilities:
  capture_requests: SUPPORTED
  capture_responses: SUPPORTED
  mock_responses: SUPPORTED
  decrypt_https: SUPPORTED
  capture_analytics: SUPPORTED
```

```yaml
strategy: system-mitm-proxy
capabilities:
  capture_requests: PARTIALLY_SUPPORTED   # Auth0 webview visible
  mock_responses: BUILD_CHANGE_REQUIRED   # Flutter backend not decrypted
  decrypt_https: BUILD_CHANGE_REQUIRED
  capture_analytics: PARTIALLY_SUPPORTED
notes: Flutter uses its own trust store; needs a QA build trusting the QA CA, or base-url redirect.
```
