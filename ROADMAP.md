# Roadmap — phase 2

The MVP ships three skills (`bootstrap-framework`, `gen-test-cases`, `implement-test-cases`) plus the Knowledge Base. Phase 2 adds two skills that make the framework self-maintaining. They are specified here, not yet built.

## `maintain-framework`

Continuously improve and keep the automation project healthy.

**Detect**
- Duplicate code and duplicate selectors
- Oversized Page/Screen/Service objects
- Unused helpers and dead tests
- Outdated locators (vs the Knowledge Base)
- Flaky tests (unstable across repeated runs)
- Architecture drift (copy-pasted flows that should be helpers)

**Refactor (behind a hard safety gate)**
- Extract common helpers, merge duplicated logic, split oversized files, update imports.
- Work on a branch or git worktree, never on the main working tree directly.
- Re-run the full suite after each change. **Never commit unless the suite is green.** If a change breaks tests and cannot be repaired safely, revert it and report.

**Health report**
- Coverage, flaky tests, technical debt, duplicated code, execution time, and prioritized recommendations.

## `sync-application`

Detect application changes automatically and target regression.

**Workflow**
1. Launch the application (or hit the API).
2. Explore the screens/endpoints the Knowledge Base knows about.
3. Diff against `application-map/`: new screens, removed screens, changed navigation, changed locators, updated permissions, new feature flags.
4. Update the Knowledge Base with the deltas.
5. Identify the test cases impacted by each change.
6. Run the impacted regression subset and report, flagging cases whose source-of-truth TC likely needs updating via `gen-test-cases`.

## Continuous automation cycle (target end state)

```
New Requirement → gen-test-cases → implement-test-cases → run → self-heal
   → update Knowledge Base → maintain-framework → regression → reporting
```

The QA engineer reviews and guides the automation strategy; the platform builds, maintains and continuously improves automation for web, mobile and API.
