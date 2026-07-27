# VGV QA Automation

An autonomous QA automation platform, packaged as a Claude Code plugin. It takes an application (web, mobile or API) from zero automation to a running, self-improving test framework.

## Core principle

One single source of truth: **human-readable, tool-agnostic test cases**.

```
Requirement  →  Test Case  →  Automation  →  Execution
```

Automation is the implementation of the test cases, never the source of truth.

## The pipeline

```
Requirements (Jira • Notion • Docs • Notes • Figma)
    → bootstrap-framework      set up the project (once per project)
    → gen-test-cases           author the test cases  (test-cases/*.md)
    → implement-test-cases     discover → implement in parallel → run → self-heal
         ↕ intercept-network    mock backend + capture traffic/analytics (as needed)
         ↕ Knowledge Base (application-map/)   read & updated every run
    → reporting
```

## Skills

| Skill | What it does | Invoke |
|-------|--------------|--------|
| `bootstrap-framework` | Interviews the QA and scaffolds a complete framework for the chosen target and stack. Runs once per project. | `/bootstrap-framework` |
| `gen-test-cases` | Generates human-readable, tool-agnostic test cases from Jira, Notion, Granola, flows or notes. The source of truth. | `/gen-test-cases [<file>] [--blank <name>]` |
| `implement-test-cases` | Reads the test cases, maps the app into a Knowledge Base, then implements and runs each case in parallel with conservative self-healing. | `/implement-test-cases [--all \| <Area> \| TC-001]` |
| `intercept-network` | Mocks backend responses (errors, empty, timeouts) and captures/asserts requests, responses and analytics events, across web/mobile/API. Detects the safest interception strategy. | `/intercept-network [mock\|capture] <description>` |

Skills also auto-trigger by intent (e.g. "set up automation for this app", "generate test cases from this ticket", "automate these TCs").

## Knowledge Base (`application-map/`)

Each project keeps a persistent, framework-agnostic knowledge store so the app is not rediscovered every run: `screens.md`, `navigation.md`, `components.md`, `permissions.md`, `endpoints.md`, `feature-flags.md`, `selectors.json`. `implement-test-cases` reads it during discovery and writes back what it learns.

## Self-healing policy

Self-healing fixes only non-functional failures (timing, waits, re-render, equivalent locators). When a failure looks functional (a screen, flow or field genuinely changed), it does not force a green result. It flags the case as a **suspected regression / app change** for human review, so real bugs are not masked.

## Install (team)

The repo is a Claude Code plugin marketplace (`.claude-plugin/marketplace.json` → marketplace `vgv-qa`, plugin `vgv-qa-automation`). Each QA runs, in their Claude Code terminal:

```
/plugin marketplace add VGVentures/automation_plugin
/plugin install vgv-qa-automation@vgv-qa
```

The four skills load and trigger by name (`/bootstrap-framework`, `/gen-test-cases`, `/implement-test-cases`, `/intercept-network`) or by intent. For a private repo, make sure `gh`/git auth has access to `VGVentures/automation_plugin`.

To update after new versions are pushed:

```
/plugin marketplace update vgv-qa
```

For local development you can instead copy each `skills/<name>/` folder into `~/.claude/skills/`.

## Roadmap

Phase 2 skills are specified in [ROADMAP.md](ROADMAP.md): `maintain-framework` (refactor, dedupe, health report) and `sync-application` (detect app changes and target regression).
