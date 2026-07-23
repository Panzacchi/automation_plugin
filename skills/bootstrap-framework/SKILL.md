---
name: bootstrap-framework
description: Use this skill when a QA wants to create a new test automation framework from scratch for a web app, mobile app or API. Triggers on phrases like "bootstrap a framework", "set up automation", "create a test framework", "start a new automation project", "scaffold tests for this app", "armar un framework de tests", "empezar automatización". Interviews the QA, assembles a framework spec, and scaffolds the project. Runs once per project; afterwards use gen-test-cases and implement-test-cases.
version: 0.1.0
allowed-tools: AskUserQuestion, Bash, Read, Write, Edit, Glob
argument-hint: "[project path or notes, optional]"
---

# bootstrap-framework

Create a complete automation framework from scratch for a web, mobile or API target. Run this **once per project**. When done, the QA continues with `gen-test-cases`, then `implement-test-cases`.

`$ARGUMENTS` may contain a project path or free notes. Use them to pre-fill answers, but still confirm the essentials via the interview.

## Core principle

The framework has **one single source of truth**: human-readable, tool-agnostic test cases in `test-cases/`. The flow is always:

```
Requirement → Test Case → Automation → Execution
```

Automation must never become the source of truth. Everything you scaffold serves that pipeline.

## Step 1 — Interview the QA

Ask with the `AskUserQuestion` tool, one concern per question, and give a short scope + example under each. Keep only the blocks relevant to the chosen target type. Read `references/framework-spec-template.md` for the full field list and the "why it matters" notes; the questions below are the interview form of that template.

**Always ask:**
1. **Objective** — framework only, or framework + automate the first scenarios. e.g. scaffold now, automate later.
2. **Target type** — web, mobile, or API (one or more). This gates the follow-ups.
3. **Project** — absolute path, name (lowercase-with-dashes), new repo vs existing, git handling.
4. **Language** — e.g. TypeScript, Java, Kotlin, Python, C#.
5. **Automation stack** — e.g. Playwright, Appium, Cypress, Detox, Supertest. If the QA is unsure, recommend one (see Step 2).
6. **Reporting** — e.g. Allure, HTML, JUnit.
7. **CI** — e.g. GitHub Actions, Azure DevOps, Jenkins, or none for now.
8. **Environments, secrets & config** — which env to target; secrets via `.env`, never committed.
9. **Architecture** — Page/Screen Object Model, service objects for API; or let you decide.

**Target-specific:**
- **Mobile** — platforms (iOS/Android), app tech (native / Flutter / React Native / hybrid; drives locator strategy), binary paths (.apk / simulator .app / .ipa), how login works (native form, in-app browser SSO, biometric, OTP).
- **Web** — base URL per environment, browsers, how login works (form, SSO/OAuth, MFA, session reuse).
- **API** — base URL per environment, protocol (REST/GraphQL/gRPC), spec/collection (OpenAPI/Postman), auth (API key / JWT / OAuth / session).

## Step 2 — Recommend a stack when the QA is unsure

- **Web** → Playwright (TypeScript).
- **Mobile** → Appium + WebdriverIO + TypeScript.
- **API** → Playwright API, or Supertest / pytest+requests depending on the chosen language.

State the recommendation and why in one line, then proceed.

## Step 3 — Scaffold the project

Create the project at the chosen path with this shape (adapt names to the stack; `pages/` for web, `screens/` for mobile, `services/` for API):

```
project/
├── src/
│   ├── pages/ | screens/ | services/
│   ├── helpers/          # centralized login, navigation, shared flows
│   ├── fixtures/
│   └── utils/
├── test-cases/           # SOURCE OF TRUTH — human-readable .md cases
├── tests/                # generated automation specs, mirrors test-cases IDs
├── application-map/      # Knowledge Base (empty for now; see below)
├── reports/
├── config/               # runner + platform configs
├── .env
├── .env.example          # documents every var, committed
├── .gitignore            # node_modules, .env, reports/, app binaries
└── <manifest>            # package.json / pom.xml / requirements.txt
```

Also:
- Install dependencies for the chosen stack and wire the chosen reporter.
- Add the CI workflow file if a CI was chosen.
- Seed an empty **Knowledge Base** at `application-map/` with placeholder files: `screens.md`, `navigation.md`, `components.md`, `permissions.md`, `endpoints.md`, `feature-flags.md`, `selectors.json`. `implement-test-cases` fills these in during discovery so the app is not rediscovered every run.
- Create a `test-cases/TEMPLATE.md` matching the `gen-test-cases` output format.
- `git init` + initial commit if the QA chose a new repo.

## Step 4 — Verify the scaffold

- Install runs clean; typecheck/build passes if the stack has one.
- A trivial smoke spec (or the runner's `--version`) executes, proving the runner and driver are wired.
- Report what was created and any default you picked on the QA's behalf.

## Step 5 — Handoff

Tell the QA to continue with:

```
gen-test-cases        # author the human-readable test cases (source of truth)
implement-test-cases  # turn those cases into running automation
```
