# Troubleshooting

Common failure modes and what they mean.

## No traffic appears at all
- The app ignores the system proxy (common with native HTTP clients) → use `base-url-redirect` / DI / QA build.
- The expected request was never triggered → re-check the UI step.
- Wrong host IP → Android emulator needs `10.0.2.2`, iOS simulator needs `127.0.0.1`.

## Only `CONNECT` tunnels appear (no decrypted content)
- The CA is not trusted by that client → wrong cert store, or Flutter's own trust store, or pinning.
- Fix: QA build trusting the QA CA, or switch to base-url redirect. Classify as `BUILD_CHANGE_REQUIRED`.

## Certificate errors in the app
- CA installed in the wrong store (user vs system), or `network-security-config` doesn't allow user CAs → add `debug-overrides` in a debug build.

## Certificate pinning
- Disable only in a QA build; never patch release. Trust one specific CA.

## Proxy ignored by some SDKs but not others
- Partial visibility (`PARTIALLY_SUPPORTED`). Webview/Auth0 traffic often visible while native/Flutter backend is not.

## Analytics not visible
- Wrong provider domain in the allowlist, or batched/gzip payloads not decoded → add the provider decoder; check `app-measurement.com` (Firebase), `*.posthog.com`, GA4 hosts.
- Provider uses a native transport that bypasses the proxy → use provider debug tooling (Firebase DebugView, GA4 DebugView, PostHog debug).

## Service Workers hide web traffic
- Set `serviceWorkers: 'block'` on the Playwright context.

## Duplicate requests / unexpected retries
- Expected in some flows; use capture assertions with counts and `times` rules to characterize rather than hide them.

## Ports already in use
- Use dynamic ports for the mock/proxy server; free them on teardown.

## Flaky network tests
- Never use fixed sleeps. Wait for a specific request/response. Reset the recorder between tests. Isolate ports and fixtures for parallel runs. Clean up mocks in teardown.
