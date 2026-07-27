# Flutter QA-build recipe

When TLS trust or backend redirection needs the app itself to change, recommend a dedicated **QA/debug build**. This is the reliable way to intercept a Flutter app's own backend traffic. It must never affect release.

## What the QA build exposes

```env
QA_NETWORK_INTERCEPTION=true
API_BASE_URL=http://10.0.2.2:4000
QA_PROXY_HOST=10.0.2.2
QA_PROXY_PORT=8000
QA_PROXY_CA_PATH=assets/certs/qa-proxy-ca.pem
```

Two independent unlocks, either is enough:

- **base-url redirect**: the app reads `API_BASE_URL` from config so it talks to your mock server (no TLS interception needed). Preferred.
- **single-CA trust**: the app's HTTP client trusts exactly one QA CA so a proxy can decrypt, via a `SecurityContext` that adds the QA cert.

## Trust exactly one CA (not "accept anything")

```dart
// QA/debug only
final context = SecurityContext(withTrustedRoots: true)
  ..setTrustedCertificates(qaCaPath);
final client = HttpClient(context: context);
```

Do NOT ship a blanket bypass. This is forbidden as a permanent solution:

```dart
badCertificateCallback = (_, __, ___) => true; // never in a shippable build
```

## Guardrails

- Enabled only in debug/QA flavors, gated by a build flag.
- Trusts only the specific QA CA, never arbitrary certificates.
- The release build must **fail** if the QA network config is still enabled.
- Document how to remove the config.
- If the app uses multiple networking stacks (dart:io + a plugin using native HTTP), apply trust to each, or prefer base-url redirect which covers all.

## Handoff

This recipe usually needs the app team. The skill's job is to detect that a build change is required, produce this exact spec, and report it as `BUILD_CHANGE_REQUIRED` with these fields, so the app team can implement it and the viability check can be re-run.
