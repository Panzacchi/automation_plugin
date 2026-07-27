---
name: gen-test-cases
description: Generate human-readable, tool-agnostic test cases in the standard TC format. Use when a QA wants to create test cases from automation flow files, Jira tickets, Notion pages, Granola transcripts, existing automation, or plain notes. Triggers on "generate test cases", "gen-test-cases", "create TCs", "write test cases from this ticket/flow/notes", "genera casos de prueba". Output is always human-readable and tool-agnostic, written to test-cases/. These files are the single source of truth the automation consumes.
version: 1.0.0
allowed-tools: Read, Write, Glob, Bash, AskUserQuestion
argument-hint: "[<file> | --blank <name>] [--non-interactive]"
---

# gen-test-cases

Generate human-readable, tool-agnostic test cases in the standard TC format. Accepts automation flow files, Jira tickets, Notion pages, Granola transcripts, or plain notes as input. Output is always human-readable and tool-agnostic.

## How to invoke

- `/gen-test-cases`: generate TCs from automation flows in the project (skip helper/assert files and utilities)
- `/gen-test-cases <file>`: generate a TC from a single source file (see Sources below for accepted types)
- `/gen-test-cases --blank <name>`: scaffold an empty TC for manual authoring
- Add `--non-interactive` to any of the above to accept defaults without prompting

`$ARGUMENTS` holds whatever the user typed after the command.

## Workflow

At the start of every batch (or single-file run), ask the user for four settings in one consolidated prompt:

1. Numbering convention: default `TC-NNN` (zero-padded, sequential). Common alternatives: per-feature prefix (`LOGIN-001`), Jira-linked (`PROJ-1234-TC1`).
2. Negative scenarios: `none` / `light` / `thorough` (see Translation rules for definitions).
3. Pre-conditions State: whether to include a State subsection by default (`yes` / `no` / `only when relevant`).
4. Conflict handling: how to treat existing TC files (`warn` / `overwrite` / `skip`). Default `warn`.

If the user passes `--non-interactive`, skip the prompt and apply defaults:

- Numbering: `TC-NNN`
- Negative scenarios: `none`
- Pre-conditions State: `only when relevant`
- Conflict handling: `warn` (even in non-interactive mode, never overwrite silently)

After generation, surface a single review summary:

- TCs that need priority review
- TCs with ambiguous State pre-conditions
- Any conflicts encountered and how they were resolved

## Sources

The skill accepts several input types. The output format and principles are identical regardless of source; only the extraction step differs.

### Automation flow files

1. Identify which flow files to process based on the invocation. Look for the flows directory convention used by the project (e.g. `flows/`, `tests/flows/`, or similar).
2. Read each flow file. Also read any helper files it references that contain expected state checks. These are expected result helpers, not test cases. Inline their checks as expected results at the point where they are referenced.
3. For each flow, output a test case in the format below.

### Jira tickets

- Derive the Goal from the ticket title and description.
- Use acceptance criteria as the basis for Steps and Expected Results.
- Always add a reference back to the ticket in the References section.

### Notion pages or Granola transcripts

- Identify the user goal first, then the start screen, then the steps in order.
- Treat any concrete values mentioned as data variables.
- If teardown isn't described, infer from what the flow mutates.
- Always add a reference back to the source page or transcript in the References section.

### Plain markdown notes

- Same approach as Notion/Granola sources.

### Blank scaffold (`--blank`)

Emit the format with placeholder text in each section so the user can fill it in manually.

## Output destination

Write all test cases to a `test-cases/` directory relative to the source location (one `.md` per TC, filename mirrors the source name without its extension). Create the directory if it doesn't exist. Report which files were written.

## Output format

```
## TC-NNN: <Human-readable flow name>

### Goal
<one sentence>

### Priority
<P0 (Critical) / P1 (High) / P2 (Medium) / P3 (Low)>

### Pre-conditions

#### Data
- <Description>: `{{variable_name}}` = `sample_value`
- <Description>: `{{variable_name}}` = `sample_value`
- <plain text for setup steps with no typed value, e.g. gallery images>

#### Start screen
<first screen the flow expects, from the leading comment or first visible assertion>

#### State (optional)
- User role: <e.g. authenticated taxpayer, admin, guest>
- Device/platform: <e.g. iOS 17+, Android 13+, any>
- Feature flags: <any required flags>
- Network: <e.g. online, offline, throttled>
- Other: <time-of-day, locale, etc.>

### Steps

| # | Action | Expected Result |
|---|--------|-----------------|
| 1 | ...    | ...             |

### Teardown
<teardown description>

### References
- <Type>: <link or identifier>
- <Type>: <link or identifier>
```

## Translation rules

### Goal

- One sentence, present tense, user perspective.
- For positive TCs: describe what the user accomplishes ("File a return and pay the amount due.")
- For negative TCs: describe what the user cannot do, or what the system prevents ("The user cannot authenticate with an invalid password.", "The user cannot submit a filing with a malformed SSN.")
- Derive from: source filename or title, leading comments, terminal screen, qualification questions, and (for negative TCs) the error state being verified.
- Describe the outcome, not the steps. If the sentence contains "tap", "enter", or "select", rewrite it.
- Include the variant in parens if relevant: "(standard path, pre-completed return)"

### Priority

- Format: `Px (Word)` where the word is one of Critical, High, Medium, Low corresponding to P0, P1, P2, P3 respectively.
- Default to P2 (Medium) unless context strongly suggests otherwise. P0 (Critical) for core monetization or auth flows, P3 (Low) for cosmetic or rarely-used features.
- After generating each TC, remind the user to review the assigned priority since automated guesses are rough.

### Pre-conditions

- Always include Data and Start screen.
- Include the State subsection based on the batch setting (yes / no / only when relevant).
- After generating the TC, ask the user to review State and add any pre-conditions Claude couldn't infer from the source.

### Actions

- App launch with cleared state: "Launch the app (cleared state)"
- Tap on a UI element: "Tap '[label]'"
- Text input: merge into the preceding tap step. "Tap '[field]' and enter '[value]'"
- Scroll, keyboard dismiss, animation waits: not standalone steps; fold into the surrounding step
- Yes/No answers to qualification questions: "Answer '[Yes/No]' to '[question text]'"

### Expected results

- A visible assertion right after an action: that action's expected result
- A referenced helper file: inline all its visible assertions as the expected result
- No explicit assertion after an action: write "Next screen loads" unless the following assertion makes the destination obvious, in that case derive from it

### Granularity

- Group multi-field forms into one step ("Fill the credit card form: number, CVV, expiry, phone, email")
- Keep one step per distinct screen transition
- Use section comments as natural step-group boundaries
- Variants that don't change the flow intent (e.g. credit card vs debit card vs ACH at the payment step) become sub-steps within the affected step, not separate TCs. Format sub-steps as a lettered or bulleted list inside the step's Action cell:
  - Example: `Tap "Payment Method" and complete payment using: a) credit card, b) debit card, c) ACH`
  - Sub-steps are only valid when all variants land on the same expected result (e.g. all three payment methods above reach the same confirmation screen). If each variant produces a different observable result, split into separate steps instead.
- Variants that change the flow intent (happy path vs declined card vs network timeout) become separate TCs.

### Negative scenarios

- Ask the user upfront whether to include negative scenarios and at what depth:
  - `none`: happy path only
  - `light`: one key validation error per flow, with different combinations for credential-like fields (e.g. wrong password with right email, right password with wrong email, malformed email format)
  - `thorough`: full coverage of invalid inputs, network failures, declined payments, permission errors, and timeout cases
- Negative TCs are always separate TCs, not branches inside the happy-path TC.

### Data

- Pull all hardcoded values from input steps and assign each a `{{snake_case_variable_name}}`
- Gallery images and other setup steps with no typed value: plain text line, no variable
- Every place a variable value appears in the steps table, replace the hardcoded value with `{{variable_name}}` so it matches the Pre-conditions Data section

### Teardown

- For flows that create persistent backend data (filings, payments, profile changes): describe the resulting clean state after reverting that data, then add "Log out of the account to return the app to an unauthenticated state."
- For flows that only mutate local app state (e.g. onboarding flag): describe the resulting clean state (e.g. "Reset the app to a clean state to restore the onboarding prompt for future runs.")
- For flows that only change session state (e.g. login): "Log out of the account to return the app to an unauthenticated state."
- For read-only flows that don't mutate any state: "None"
- Never reference the automation tool, framework commands, or mechanism. Describe the resulting state, not how to achieve it.

### References

- Include this section only if the TC has linked artifacts. Omit entirely otherwise.
- One bullet per reference. Format: `<Type>: <identifier>`. Examples:
  - `Source: JIRA-1234`
  - `Source: Notion page "Q2 Login Redesign" (2026-04-15)`
  - `Source: Granola transcript, QA grooming 2026-05-10`
  - `Spec: Figma file ABC123`
  - `Flow file: flows/easyFileAndPay.yaml`
- When the TC is generated from a source file or ticket, always add it here so the trail back to the requirement is preserved.

### Conflict handling on regenerate

- Before writing a TC file, check whether `test-cases/<name>.md` already exists, or whether a TC with a very similar Goal or filename exists in the directory.
- Apply the batch's conflict handling setting:
  - `warn`: show the user and ask per-conflict (overwrite / skip / new filename)
  - `overwrite`: replace existing TC, but log it in the final review summary
  - `skip`: leave existing TC untouched, log it in the final review summary
- Never overwrite silently regardless of setting. The user gets a summary at the end.

## Output principles

- Test cases are written for humans and must be tool-agnostic.
- Describe what a tester does and observes, not how an automation framework executes it.
- Never reference any automation tool, framework syntax, command names, or assertion APIs in the output.
- The source is the input; the test case is a translation, not a transcription.

## Network scenario conventions

When a case involves the backend or analytics, phrase it in plain, tool-agnostic language (the `intercept-network` skill implements it later). Keep the wording consistent:

- Mocked error, as a Pre-condition: "The backend returns HTTP 500 for POST /payments."
- Empty state, as a Pre-condition: "The backend returns an empty list for GET /transactions."
- Analytics, as an Expected Result: "The analytics event `payment_started` is sent with `payment_method` set to `card`."
- Negative traffic, as an Expected Result: "No payment request is sent until the user confirms the operation."

Never name a proxy, mock library or framework API — describe the backend behavior and what the tester observes.
