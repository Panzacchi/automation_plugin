# Interception contract

A single language-agnostic contract that every adapter implements, so specs read the same whether the engine is Playwright routing or a mockttp proxy. Implement it as real shared code in the project (e.g. `src/network/`), not just docs.

```ts
interface NetworkInterceptor {
  start(): Promise<void>   // begin intercepting (register routes / launch proxy)
  stop(): Promise<void>    // stop and restore all environment state
  reset(): Promise<void>   // clear rules + recorded traffic between tests

  mock(rule: MockRule): Promise<void>
  capture(filter: TrafficFilter): Promise<TrafficRecorder>
  waitForCall(assertion: TrafficAssertion): Promise<CapturedCall>
}
```

## Mock model

```ts
type MockRule = {
  match: {
    method?: string
    url?: string | RegExp
    hostname?: string | RegExp
    headers?: Record<string, unknown>
    query?: Record<string, unknown>
    body?: unknown
  }
  action:
    | { type: 'fulfill'; status: number; headers?: Record<string, string>; body?: unknown }
    | { type: 'abort'; reason?: string }
    | { type: 'delay'; milliseconds: number }
    | { type: 'modify-response'; transform: string }  // named transform applied to the real response
  times?: number   // apply at most N times, then fall through
}
```

## Capture model

```ts
type TrafficFilter = {
  methods?: string[]
  urls?: Array<string | RegExp>
  hostAllowlist?: Array<string | RegExp>
  hostBlocklist?: Array<string | RegExp>
  includeRequests?: boolean
  includeResponses?: boolean
  includeBodies?: boolean
}

interface TrafficRecorder {
  calls(): CapturedCall[]
  clear(): void
}

type TrafficAssertion = {
  method?: string
  url?: string | RegExp
  hostname?: string | RegExp
  bodyMatches?: unknown
  timeoutMs?: number
  expectAbsent?: boolean   // assert the call did NOT happen within timeout
}
```

## Lifecycle & isolation rules

- `start()` before navigation / app launch so nothing is missed.
- `reset()` in `beforeEach`/`afterEach` so rules and recordings never leak between tests.
- `stop()` in global teardown restores proxy/cert/device state.
- One-time mocks use `times: 1`; detect and warn about configured-but-unused mocks at teardown.
- Assertions wait for a specific call (`waitForCall`), never a fixed sleep.
- For parallel runs, isolate ports and recorders per worker.
