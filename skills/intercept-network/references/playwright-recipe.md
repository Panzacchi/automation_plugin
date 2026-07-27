# Playwright recipe (web, `runner-native`)

Playwright's built-in interception is the most stable strategy: no certificates, easy per-test isolation, works in parallel. Use it for any Playwright project.

## Suggested framework layout

```
src/
├── fixtures/
│   └── network.ts            # exposes `network` fixture implementing NetworkInterceptor
├── network/
│   ├── network-recorder.ts   # request/response capture
│   ├── network-matcher.ts    # MockRule / TrafficFilter matching
│   ├── analytics-recorder.ts # provider filters + decoders
│   └── redaction.ts          # strip sensitive headers/fields
test-data/
└── network/fixtures/         # canned JSON responses
```

## APIs to wrap

- Mock: `browserContext.route()` (whole test) or `page.route()` (page-specific) → `route.fulfill()`, `route.abort()`, `route.fetch()` (for modify-response).
- Capture: `page.on('request')`, `page.on('requestfinished')`, `page.on('response')`, `page.waitForResponse()`.

Prefer `browserContext.route()` for rules shared across the test, `page.route()` for page-specific ones.

## Expected test shape

```ts
test('shows payment error', async ({ page, network }) => {
  await network.mock({
    match: { method: 'POST', url: '**/payments' },
    action: { type: 'fulfill', status: 500, body: { code: 'PAYMENT_FAILED' } },
    times: 1,
  })
  await page.getByRole('button', { name: 'Pay' }).click()
  await network.expectRequest({ method: 'POST', url: '**/payments' })
  await expect(page.getByText('Payment could not be completed')).toBeVisible()
})
```

## Analytics

```ts
await network.captureAnalytics({ provider: 'posthog', hostAllowlist: ['*.posthog.com'] })
await page.getByRole('button', { name: 'Pay' }).click()
await network.expectAnalyticsEvent({
  provider: 'posthog',
  name: 'payment_started',
  properties: { payment_method: 'card' },
})
```

## Considerations

- Register routes **before** navigation.
- Remove routes in teardown; never share mock state between tests.
- Support one-time mocks (`times`) and detect configured-but-unused mocks.
- Block Service Workers when they hide traffic (`serviceWorkers: 'block'` in context).
- Routing may disable HTTP caching — note it.
- HAR (`routeFromHAR`) is a valid secondary strategy for record/replay.
- Filter third-party beacons (GA4, Socure, Cookiebot, TruValidate in pw-framework) via host allow/blocklists.
- Isolate ports/fixtures for parallel execution.
