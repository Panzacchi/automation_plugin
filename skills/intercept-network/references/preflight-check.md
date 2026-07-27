# Preflight check

Run before any mock or capture. Produces the capability matrix and a viability report. Never skip on mobile.

## 1. Detect the stack

- **Platform**: web / Android / iOS / API.
- **Runner**: Playwright, Cypress, WebdriverIO+Appium, pytest, etc. (read `package.json`, config files).
- **Language**: TS/JS, Java/Kotlin, Python, C#.
- **App framework** (mobile): native, Flutter, React Native, hybrid. Flutter and RN often use their own HTTP/trust stacks.
- **HTTP clients**: fetch/XHR (web), dart:io/Dio/http (Flutter), OkHttp/URLSession, etc.
- **Config mechanisms**: `BASE_URL`/`API_BASE_URL` env, build flavors, DI seams.
- **Proxy support**: does the app honor the system proxy?
- **Certificates**: system vs user vs app-trusted CA; network-security-config; pinning.
- **QA build**: is a debug/QA flavor available that can trust a CA or point at a mock server?

## 2. Choose the strategy (first viable wins)

`runner-native` → `base-url-redirect` → `dependency-injection` → `explicit-proxy` → `system-mitm-proxy` → `provider-observability` → `unsupported`.

For web this is almost always `runner-native`. For mobile prefer `base-url-redirect`/`dependency-injection` (needs a QA build) because they avoid the TLS trust wall; use `system-mitm-proxy` only when nothing better exists, and only after the viability check passes.

## 3. Mobile viability check (mandatory)

Three phases, each gates the next:

- **Phase 1 — connectivity**: device → proxy on a controlled plain-HTTP URL. Confirms the device routes through the proxy at all.
- **Phase 2 — controlled TLS**: device → proxy → a controlled HTTPS endpoint. Confirms the proxy CA is trusted for the transport under test.
- **Phase 3 — real app request**: trigger a known in-app action and observe whether the proxy sees a connection, only a `CONNECT` tunnel, decrypted content, some SDKs but not others, or nothing.

## 4. Classify the result

- **No connection observed** → app ignores proxy / direct connection / request not triggered → `INCONCLUSIVE` or `ENVIRONMENT_CHANGE_REQUIRED`.
- **`CONNECT` seen but not decrypted** → CA not trusted / pinning / custom trust store / wrong cert store → `BUILD_CHANGE_REQUIRED`.
- **Traffic decrypted** → `SUPPORTED`.
- **Partial** (e.g. Auth0 Custom Tab visible, Flutter backend not; Firebase yes, Datadog no) → `PARTIALLY_SUPPORTED`.

## 5. Emit the viability report

State the selected strategy, the capability matrix, and for anything below `SUPPORTED` the exact build/env change that would unlock it (see `flutter-debug-build-recipe.md`). Then, and only then, proceed to implement mocks/captures for the capabilities that are `SUPPORTED`.
