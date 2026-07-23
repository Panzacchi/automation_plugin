---
name: implement-test-cases
description: Use this skill to turn human-readable test cases into running automated tests for a web app, mobile app or API. Triggers on phrases like "implement the test cases", "automate these TCs", "implement-test-cases", "write the automation for these cases", "run and self-heal the specs", "automatiza los casos de prueba", "implementá los TCs". Reads test-cases/, discovers the app, maps locators into the Knowledge Base, then implements and runs each case in parallel with conservative self-healing.
version: 0.1.0
allowed-tools: Agent, Bash, Read, Write, Edit, Glob, AskUserQuestion
argument-hint: "[--all | <Area> | TC-001]"
---

# implement-test-cases

Transform business documentation (the test cases) into executable automation. Test cases in `test-cases/` are the **single source of truth**; specs are their implementation, never the other way around.

`$ARGUMENTS` scopes the run:
- `--all` — every pending test case
- `<Area>` — one area folder, e.g. `Payments`, `Login`
- `TC-001` — a single case by id
- empty — ask the QA what to implement

## Step 0 — Discover work

1. List `test-cases/**/*.md` and the specs already in `tests/`.
2. Map each TC to a spec by **id** (e.g. `test-cases/Login/TC-001.md` → `tests/login/TC-001.spec.*`).
3. Compute the pending set (TCs with no green spec yet), filtered by `$ARGUMENTS`.
4. Read the target and stack from the project (config, `.env`, `package.json`/manifest). Confirm the live target is reachable (emulator booted / dev server up / API reachable). If not, ask the QA to start it.

If nothing is pending, report and stop.

## Step 1 — Discovery (serial, foreground)

Runs once, before any parallel work, to remove the shared-object collision hazard.

Drive the live target to understand only what the pending TCs touch, and build/update the **Knowledge Base** at `application-map/`:
- `screens.md` — screens/pages, how to reach them, key elements
- `navigation.md` — flows and transitions
- `components.md` — reusable components
- `permissions.md` — roles / gating
- `endpoints.md` — API surface (when applicable)
- `feature-flags.md`
- `selectors.json` — stable locator catalogue keyed by screen/element

Read the Knowledge Base first and only explore what is missing or stale, so the app is not rediscovered every run. If a `map-app` / `app-knowledge-base` artifact exists, seed from it.

Then create or extend the **shared objects once**: Page Objects (web), Screen Objects (mobile), Service Objects (API), plus the centralized `helpers/` (login, navigation), `fixtures/` and env config. These are the shared surface, so build them here serially, not inside the parallel workers.

Commit the Knowledge Base + shared objects before fan-out so workers build on a stable base.

## Step 2 — Parallel implementation (subagents)

Spawn one worker per pending TC with the `Agent` tool, bounded concurrency (about 4-6 at a time). Give each worker: the TC file path and id, the target/stack, the Knowledge Base paths, and the conventions below. Each worker:

1. Reads its one TC.
2. Writes **its own** spec file (`tests/<area>/TC-xxx.spec.*`) against the already-built shared objects. It does not edit shared objects; if it finds a gap, it records it and reports back rather than editing shared files concurrently.
3. Runs its spec against the live target.
4. Self-heals per the policy below and re-runs.
5. Returns: status (green / suspected-regression / blocked), the spec path, and any shared-object gap it hit.

Use worktree isolation only if workers must touch shared files; the "own spec file only" rule normally avoids that. After the batch, apply any reported shared-object gaps serially, then re-run affected specs.

## Conventions (enforce in every spec)

- Spec `describe`/id mirrors the TC id and title.
- Locators: stable selectors first — accessibility id / `testID` / `data-testid` / id. XPath only as a last resort, and never brittle absolute paths.
- Reuse centralized login + navigation helpers, fixtures and env config. No copy-pasted flows.
- Tests are idempotent: self-clean any data they create, and respect backend limits (rate/quota).
- Data values come from the TC's Pre-conditions / `.env`, never hardcoded secrets.

## Self-healing policy (conservative)

Auto-fix and re-run only for **non-functional** causes:
- timing / explicit waits, element not ready, re-render races
- an equivalent locator for the same element (same screen, same intent)
- transient navigation / retry

If the failure signature looks **functional** — a screen, flow, field or expected result genuinely changed vs the TC and the Knowledge Base — **stop, do not force green**. Record the TC under a **"suspected regression / app change"** review list with what differs. This is often a real bug or an app change that should update the source-of-truth TC, not something to silently heal.

Never mark a TC done unless its spec ran green for non-functional reasons.

## Step 3 — Report

Aggregate and surface:
- Green: TC id → spec path.
- Suspected regression / app change: TC id → what differed (needs human review).
- Blocked: TC id → reason (e.g. shared-object gap, target unreachable).
- Execution report + failure screenshots (via the project reporter).
- Knowledge Base and Page/Screen/Service objects updated in Step 1.

Remind the QA that suspected-regression items may mean the app changed and the corresponding test case (source of truth) should be updated via `gen-test-cases`, not just the spec.
