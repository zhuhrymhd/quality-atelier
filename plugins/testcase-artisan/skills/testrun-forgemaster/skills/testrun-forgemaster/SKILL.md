---
name: testrun-forgemaster
description: >
  Writes and runs automated test code (XCTest, XCUITest, Espresso, JUnit, Jest,
  Vitest, Playwright, pytest, go test, etc.) strictly from test case specs already
  defined in docs/test-cases/, then reports results in plain language. Use when the
  user wants tests written as code, executed, or verified: "run the smoke tests",
  "write the UI test for SSO", "does login still pass after the refactor", "run the
  unit tests for checkout", or a bare "make tests for X" / "test this feature" when
  no test case spec is being requested. Only writes tests, runs them, and reports;
  never modifies app code. If no spec exists for the feature, it recommends running
  testcase-artisan first. Do NOT use for writing or updating test case specs or
  scenario lists - that is testcase-artisan's job.
---

# Testrun Forgemaster

Turns test case specs into test code, runs it, and reports what happened. Read this
whole file before doing anything - the steps depend on each other.

## Boundary

This skill does three things and nothing else:

1. Writes test code that implements test cases already defined in `docs/test-cases/`.
2. Runs tests.
3. Reports results, and may suggest fixes.

It must never:

- Modify application code, including adding accessibility identifiers, test hooks,
  or dependency injection seams. If a test needs one, suggest it in the report.
- Weaken an assertion, add a retry, or raise a timeout just to make a failing test
  pass. A failing test is information, not a problem to hide.
- Write to `docs/test-cases/` or `.quality-atelier.json` beyond its own section.
  Test case specs belong to testcase-artisan.
- Edit, rename, or delete test code it didn't write. Test functions without a
  `Covers:` comment are the user's and stay untouched, even if they look redundant.
- Add dependencies, create test targets, or edit build files (`project.pbxproj`,
  `build.gradle`, `package.json`, etc.). If one is needed, ask the user.
- Apply its own suggested fixes, including fixes to test code it wrote. Suggest, then
  stop. The user decides.

It only implements what the spec says. If the user asks for a test that isn't
described in `docs/test-cases/`, don't improvise one.

## Step 1: Load configuration

Read `.quality-atelier.json` at the repo root and use the `testrun-forgemaster`
section. The schema is in `references/config.md`.

If the file or this skill's section for the detected platform doesn't exist, ask
the user once for the required values (the platform pack lists which ones), save
them, and tell the user where they were saved and that they can edit that file to
change them later. Don't ask again on later runs.

## Step 2: Detect the platform

Use the same marker files as testcase-artisan:

- `*.xcodeproj`, `*.xcworkspace`, `Package.swift` → iOS
  (`references/platforms/ios.md`)
- `AndroidManifest.xml` present → Android (`references/platforms/android.md`)
- `package.json` with a frontend framework, or `vite.config.*` / `index.html` at the
  root → Web (`references/platforms/web.md`)
- A server framework and no frontend framework, or an OpenAPI/Swagger file →
  Backend/API (`references/platforms/backend-api.md`)

Load the matching pack before writing or running anything. It defines the test
frameworks, file locations, run commands, and platform-specific uncertainty
triggers. If no pack matches, tell the user this platform isn't supported yet and
stop.

## Step 3: Determine mode and scope

**Mode**, from what the user asked:

- Wants test code written ("write the tests for SSO") → **write**.
- Wants tests run or results checked ("run the smoke tests", "does login still
  pass") → **run**.
- Asks for both explicitly ("write and run the SSO tests") → **write, then run**
  only what was just written.
- Ambiguous ("make tests for login", "test this feature") → **write**, then end by
  asking whether to run them. Don't run without an explicit yes.

**Scope**, narrowest that matches the request:

- Specific test case IDs
- One feature (`docs/test-cases/<feature>.md`, plus any split files like
  `<feature>-<subflow>.md`)
- A subset by flag or level: `Smoke: true`, or one `Level test` value
- Everything, only when the user clearly asks for it

## Step 4: Read the specs

1. Open the spec file(s) in scope. Field meanings are in
   `references/field-format.md`, a copy of testcase-artisan's contract.
2. If the spec file doesn't exist, or it exists but doesn't describe the behavior
   the user asked about, stop. Tell the user no test case spec covers this yet and
   recommend running `/testcase-artisan` for that feature first. Don't write tests
   from docs, code, or your own reading of the feature.
3. Sort each test case in scope by `Status`:
   - `active` → normal.
   - `needs-review` → implement and run, but flag it (Step 5 and Step 7).
   - `superseded-by` → don't write new code for it, don't run existing code for it.

## Step 5: Write mode

1. Find existing coverage: search the platform's test locations for `Covers:`
   comments and build a map from test case ID to test function.
2. For each test case in scope that is `active` or `needs-review` and has no
   existing function, write one test function:
   - At the `Level test` given in the spec. Don't change it. If another level seems
     better, keep the spec's level and record the reason as a suggestion for the
     report.
   - Implementing the spec's `Precondition`, `Steps`, and `Expected` literally. Every
     `Expected` statement becomes an assertion.
   - With a traceability comment directly above the function, using the language's
     comment syntax: `// Covers: TC-LOGIN-007` (Python: `# Covers: TC-LOGIN-007`).
     This is mandatory on every function this skill writes, no exceptions.
   - For a `needs-review` test case, add a second line under it:
     `// NEEDS-REVIEW: TC-LOGIN-003 flagged in docs/test-cases/login.md`
   - In the file and location the platform pack specifies, following the project's
     existing structure first.
3. For test cases that already have a function: leave the code as it is. If the
   spec is now `needs-review` and the marker comment is missing, add only the marker
   comment. If it is now `superseded-by`, add
   `// SUPERSEDED: replaced by TC-LOGIN-018` above it and suggest deleting it in the
   report. Don't delete it.
4. Before writing anything, check every test case in scope against the uncertainty
   triggers below and in the platform pack. Collect all questions and ask them in
   one message. Write the test cases that have no open question; hold the rest
   until the user answers.
5. Follow `references/writing-style.md` for comments, test names, and failure
   messages.

### Uncertainty triggers (ask, don't guess)

- The test must drive a third-party screen the app doesn't control (Google or Apple
  sign-in sheet, a payment provider page, an OAuth web view). Ask whether to mock at
  the app's auth or network boundary, or use a dedicated test account.
- A UI element the test must find has no stable identifier (accessibility
  identifier, test tag, `data-testid`, or accessible role and label).
- An integration test needs to mock network or storage, but the app has no seam to
  inject a mock through.
- The spec's `Steps` or `Expected` can't be turned into concrete actions or
  assertions without inventing details.
- The test needs real credentials, a real account, or a real external service.
- The test target, test directory, or test framework the platform pack expects
  doesn't exist in the project.

## Step 6: Run mode

1. Select the test functions for the test cases in scope using the `Covers:` map.
   Skip `superseded-by`. If an in-scope test case has no function yet, don't write
   one silently; list it in the report under "Not covered yet".
2. Confirm before running when the selection contains any `e2e-multi-screen` test,
   or more than 5 `ui-single-screen` tests. Say how many tests of which level, that
   it involves a build and a simulator, emulator, or browser, and that it may take
   several minutes. Don't state a precise duration. `unit`, `integration`,
   `snapshot`, and `contract` tests run without asking.
3. Run only the selected tests, using the platform pack's filtered run command, not
   the whole suite. For backend tests, apply the platform pack's test-environment
   safety check before running anything.
4. If the run reports zero tests executed, the filter is wrong. Report that, and
   never describe it as a pass.
5. Save the full raw output to `.test-results/<YYYY-MM-DD-HHMM>-<scope>.log`. If
   `.test-results/` isn't in `.gitignore`, suggest adding it in the report (don't
   edit `.gitignore` yourself).

## Step 7: Report

Write the report in the format defined in `references/report-format.md`. The short
version: summary line first, then failures mapped to test case IDs and the step that
failed, then needs-review results kept separate from regressions, then skipped and
uncovered items, then suggestions, then the raw log path. Never dump raw tool output
into the chat.

When something fails, describe what was expected and what actually happened. If you
have a theory about the cause, label it as unverified. Suggest a fix if you have
one, then stop.

## Reference files

- `references/field-format.md` - the test case spec contract (copy of
  testcase-artisan's; keep in sync).
- `references/writing-style.md` - writing rules (copy of testcase-artisan's).
- `references/report-format.md` - report structure for write and run modes.
- `references/config.md` - `.quality-atelier.json` schema and first-run flow.
- `references/platforms/ios.md`, `android.md`, `web.md`, `backend-api.md` - test
  frameworks, level-to-framework mapping, file locations, run commands, and
  platform-specific uncertainty triggers.
