# Mobile proxy recipe (`system-mitm-proxy`)

Fallback for when `base-url-redirect` / `dependency-injection` aren't available. Route device traffic through a local proxy and trust its CA. **Gated by the viability check** — many apps (especially Flutter) won't trust the CA, so verify before promising anything.

## Proxy engine

- **mockttp** (primary, TS): programmable rules, HTTP+HTTPS, generates its own CA, integrates with WDIO. Use for both mock and capture.
- **mitmproxy** (alternative): manual inspection, advanced diagnostics, complex rules, protocol debugging.

## Android (emulator)

```bash
# point the device at the proxy (host reachable at 10.0.2.2 from the emulator)
adb shell settings put global http_proxy 10.0.2.2:8000

# install the proxy CA (user store)
adb push qa-proxy-ca.pem /sdcard/
# then Settings > Security > Encryption & credentials > Install a certificate > CA
# (or push into the system store on a rooted/emulator image)

# restore afterwards
adb shell settings put global http_proxy :0
```

Cover: host IP resolution (`10.0.2.2`), user vs system vs app-trusted CA differences, `network-security-config.xml` `debug-overrides` (needed for apps to trust user CAs on API 24+), certificate pinning, and apps that ignore the system proxy entirely.

## iOS (simulator)

- Set the proxy in the simulator's network settings (or via the host's network config).
- Import the CA and enable full trust in the simulator keychain.
- Watch for ATS restrictions and pinning.
- Restore settings after the run.

## Physical devices

Same idea but CA trust and proxy config differ per OS version; document per-device. Prefer a QA build over device-level MITM when possible.

## Teardown (mandatory)

Always restore the device proxy (`:0` on Android), remove the temporary CA, stop the proxy, and delete temporary files. Leaving a device proxied or a CA installed is a security and reliability hazard.

## Flutter caveat

Flutter's Dart HTTP client uses its own compiled-in trust store, so even a correctly installed system/user CA is typically not trusted by app traffic. If the viability check shows `CONNECT` tunnels without decryption, this is why — switch to `configurable-backend-recipe.md` or `flutter-debug-build-recipe.md`. Traffic that flows through platform webviews (e.g. the Auth0 Custom Tab) does honor the system CA and can be captured.
