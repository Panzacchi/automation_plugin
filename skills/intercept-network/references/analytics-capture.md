# Analytics capture

Capturing and asserting analytics is a first-class use case. Analytics are just outbound requests to provider domains, so the same capture engine records them; the extra work is decoding provider payloads and being honest about semantics.

## Providers (initial adapters)

Firebase Analytics, GA4, PostHog, Datadog, Customer.io. The design allows more later. Each provider is a domain filter + a payload decoder.

Typical domains: `*.posthog.com`, `*.google-analytics.com` / `*.analytics.google.com` (GA4), `app-measurement.com` (Firebase), `*.datadoghq.com`, `track.customer.io` / `*.customer.io`.

## Event assertion model

```ts
type AnalyticsEventAssertion = {
  provider?: string
  name: string
  properties?: Record<string, unknown>
  ignoreProperties?: string[]
  timeoutMs?: number
}
```

## Decoding

Payloads arrive in different encodings; support:

```
json | form | text | gzip-json | custom
```

Also handle **batched** payloads (many events in one request — GA4, PostHog and Firebase all batch) by iterating events inside the body, and provider-specific `custom` decoders where needed.

## Verification semantics (be precise)

Distinguish and only claim what you actually observed:

```
event_created         the app built the event (not observable from the network)
event_queued          buffered client-side (not observable)
event_request_sent    the outbound request left the device  ← what a network assertion verifies
event_server_accepted the provider accepted it (needs provider APIs, not the network)
```

A network assertion verifies `event_request_sent`. Do not claim the provider processed or accepted the event.

## Dynamic data

Ignore volatile properties by default so assertions aren't flaky: `timestamp`, `session_id`, `device_id`, `request_id`, `anonymous_id`, `correlation_id`, build timestamps. Match only the meaningful properties (e.g. `payment_method: card`) and allow partial matching.
