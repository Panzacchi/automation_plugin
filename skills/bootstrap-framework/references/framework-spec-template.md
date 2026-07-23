# Prompt: bootstrap a test automation framework

Copy this file, fill it in, and hand it back to me. I will use it to stand up a
test automation framework (mobile app, web, or API) and — if you ask — automate
the first scenarios end to end.

**How to use it**
- Fill every `<…>` placeholder.
- In section 3, keep only the block(s) for the target type(s) you are automating and delete the rest.
- Leave a section blank only if you truly have no preference — I will pick a sensible default and call it out.
- Each section ends with a short *Why it matters* note (lessons from past builds); those are for you, you don't need to answer them.

---

## 1. Objective
- What I want built: <e.g. a fresh automation framework + the first N scenarios automated end-to-end>
- Target type(s): <mobile app | web | API> (keep the matching block in section 3)
- Definition of "done" for this run: <e.g. framework scaffolded, 1 smoke scenario green, report generated>

## 2. Project location & naming
- Repo/folder path: <absolute path, new or existing>
- Project name: <name>
- New repo or add to existing? <new | existing; if existing, describe what's there>
- Git: <init + commits? branch? or leave uncommitted>

## 3. Target under test  (keep only the relevant block)

### 3a. Mobile app
- Platforms: <iOS | Android | both>
- Binaries: <paths to .apk / .app-simulator / .ipa, or "I'll provide later">
- App tech: <native | Flutter | React Native | hybrid webview> (affects locator strategy)
- Bundle id / package: <if known>
- Devices/emulators: <target device names + OS versions>
- Auth/login: <how login works — native form, in-app browser / SSO, biometric, OTP>

### 3b. Web
- App URL(s): <base URL per environment>
- Browsers: <chromium | firefox | webkit | all>
- Auth/login: <form, SSO/OAuth, MFA, session reuse>
- Responsive/mobile-web scope: <yes/no, viewports>

### 3c. API
- Base URL(s) per environment: <…>
- Protocol/style: <REST | GraphQL | gRPC>
- Spec available? <OpenAPI/Swagger/Postman collection path or URL, or none>
- Auth: <api key | bearer/JWT | OAuth client-credentials | session>
- Contract/schema validation wanted? <yes/no>

> **Why it matters (auth):** in-app-browser / SSO logins behave very differently per platform. On Android a Chrome Custom Tab exposes a webview context to automation; on iOS the Safari view often does not. Flag how login works early — it's usually the trickiest part.

## 4. Tech stack & tooling  (leave blank to let me recommend)
- Language: <…>
- Test runner / framework: <…>
- Automation driver/library: <…>
- Assertion library: <…>
- Package manager / runtime versions: <…>
- Any mandated tools or bans: <…>

## 5. Test cases — the source of truth
- Do you have written test cases? <attach / point to folder or files, or "generate from exploration">
- Where they live: <path/URL — folder of files, spreadsheet, test-management tool (TestRail/Xray/Zephyr), etc.>
- **How the scenarios are written** (so I know how to read and lift them): <pick one and give a real sample>
  - Gherkin / BDD: `Feature` + `Scenario` with `Given/When/Then` (`.feature` files) → I generate step definitions and reuse steps
  - Step-by-step table: numbered `Action | Expected Result` rows → I translate each row to spec code
  - Plain prose / checklist: free-text narrative → I structure it before automating
  - Formal template: named fields (ID, Goal, Priority, Pre-conditions/Data/State, Steps, Teardown)
  - Other: <describe>
- Include a sample of ONE real case so I match the exact shape and wording.
- Case ID convention: <e.g. AREA/NN-SHORT-NAME>
- Mapping rule: <e.g. one spec per case, same ID in the describe/Scenario; or one .feature per area>
- If Gherkin: <shared step library expected? tags for smoke/regression? scenario outlines with examples?>

> **Why it matters:** keeping cases in a written source (markdown, Gherkin, a tool) decoupled from the code makes them reviewable by non-devs, and each case maps 1:1 to a spec by ID. Knowing the writing style up front decides how scenarios are lifted: Gherkin needs a runner + step definitions and a shared step library; step-by-step tables translate row-by-row into spec code; prose must be structured first. Always give one real sample so the shape and wording are matched exactly.

## 6. Architecture & conventions  (state preferences or leave to me)
- Structure pattern: <e.g. Page/Screen Object Model, service objects for API>
- Locator strategy priority: <e.g. accessibility id > testID > text; for web: data-testid > role > css>
- Shared flows/helpers to centralize: <e.g. login, navigation, data setup>
- Folder layout expectations: <config / cases / specs / screens|pages|services / helpers>
- Coding style to match: <link to an existing repo/style, or none>

> **Why it matters (locators):** prefer stable semantic locators. For Flutter/native there are no resource-ids, so accessibility labels are the anchor; some characters (`$`, `•`) break query parsers and need XPath. Centralizing login/navigation into shared flow helpers keeps specs short and resilient.

## 7. Test data, secrets & environments
- Environments: <dev | qa | staging | prod and which to target>
- Config mechanism: <.env, config files — and what's committed vs secret>
- Credentials/test accounts: <how provided; never commit real secrets>
- Data that mutates state: <can tests create/modify real data? cleanup expected? daily/rate limits?>
- Idempotency requirement: <should every scenario be repeatable and self-cleaning?>

> **Why it matters (idempotency):** tests that mutate real data must self-heal (pre-clean any residue from a prior interrupted run) and clean up in teardown, and respect backend limits (e.g. "max 2 payments per day"). Otherwise the second run fails on leftover state.

## 8. Reporting & CI
- Reporter: <e.g. Allure, HTML, JUnit; auto-generate after run?>
- Artifacts on failure: <screenshots, logs, network/HAR, video>
- CI target: <GitHub Actions / other, or local only for now>

## 9. Scope of THIS run
- Build framework only, or framework + automate scenarios? <…>
- Which scenarios to automate now (by ID/priority): <…>
- What is explicitly out of scope for now: <…>

## 10. How to discover locators / endpoints
- Can you run the target so I can explore it live? <emulator up / dev server / API reachable>
- Preferred discovery approach: <inspect running app, read the codebase, use the spec/collection>
- Known gotchas: <flaky screens, modals, rate limits, feature flags, third-party auth>

> **Why it matters:** the fastest path to correct locators is driving the running target and dumping the tree/DOM, not guessing. If you can have the emulator booted / dev server up / API reachable, the build goes much faster and the locators are real.

## 11. Constraints & guardrails
- Time/scope limits, must-not-touch areas, security/authorization notes: <…>
- Anything the framework must NOT do (e.g. hit prod, send real payments over $X): <…>

## 12. Existing skills / MCP / reusable assets to leverage
- Skills I should use during the build: <name them, or "use whatever fits">
  - Discovery/mapping: <e.g. `map-app`, `app-knowledge-base` to map screens, locators, flows into a persistent map>
  - Scenario generation: <e.g. `api-test-scenario-generator` for API scaffolds; other generators>
  - Validation/audits: <e.g. `validate-accessibility-ids`, `validate-wcag-aa`>
- MCP servers available for this target: <e.g. a docs MCP for the library, a mobile/browser control MCP, an API/collection source>
- Existing repos/frameworks to reuse patterns from: <path/URL, or "none — start fresh">
- Persisted knowledge to read/update: <e.g. an APP_MAP.md / knowledge-base file to build on across runs>

> **Why it matters:** naming the available skills and MCP servers up front lets me offload the expensive parts to purpose-built tools instead of hand-rolling them: app-mapping skills accelerate locator/flow discovery and persist a reusable map; scenario generators scaffold cases; validation skills add accessibility/WCAG audits for free. Pointing me at an existing framework or knowledge file means I extend it instead of starting cold.

## 13. Verification I expect
- How you'll prove it works: <run the smoke scenario green, attach report/screenshots, show the run output>
