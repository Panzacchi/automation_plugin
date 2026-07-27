# Configurable backend recipe (`base-url-redirect`)

The most reliable mobile strategy and a great option for web/API too: point the app at a local mock server via config, so no proxy or CA is involved. Requires a build/config that lets you override the API base URL.

## Redirect the app

```env
# web / node
API_BASE_URL=http://127.0.0.1:4000

# Android emulator reaches the host via 10.0.2.2
API_BASE_URL=http://10.0.2.2:4000

# iOS simulator reaches the host via localhost
API_BASE_URL=http://127.0.0.1:4000
```

For coign, this depends on the QA build exposing a configurable base URL (see `flutter-debug-build-recipe.md`). Confirm with the app team.

## Mock server options

- **mockttp** (recommended, TS): programmable rules, HTTP+HTTPS, generates a CA, integrates with the test lifecycle. Good default for a Node/TS stack.
- **WireMock**: JVM, mature, stateful stubs and scenarios.
- **JSON Server / a tiny Express app**: quickest for static canned responses.

## Lifecycle

- Start the mock server on a **dynamic port** (avoid collisions in parallel/CI); pass the URL to the app via env.
- Expose a **health check** and wait for it before the test starts.
- Load canned responses from `test-data/network/fixtures/`.
- Support response sequences and `times` so a rule can change across calls.
- Tear the server down and free the port after the run.

## Why prefer this on mobile

It sidesteps the Flutter TLS trust wall entirely: the app talks plain HTTP (or HTTPS with a URL you control) to your mock, so there is nothing to decrypt and no CA to install. The trade-off is it needs app cooperation (a build that reads the base URL from config) and it replaces the backend rather than observing the real one, so pair it with `capture` strategies when you need to assert real traffic.
