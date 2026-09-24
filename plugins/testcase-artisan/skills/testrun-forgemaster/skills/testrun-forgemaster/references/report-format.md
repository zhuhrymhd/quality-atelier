# Report Format

Reports are read by a person first. Lead with the outcome, map everything back to
test case IDs and titles, and keep raw tool output out of the chat. Omit any section
that would be empty.

## Run mode

```
Smoke tests (iOS): 3 passed, 1 failed, 1 needs review (48s)

FAILED
  TC-LOGIN-007  Login via Google SSO
  Failed at step 2: "Pick an account in the Google sheet"
  Expected: account picker appears
  Actual: no element matched "Google account picker" within 10s
  Location: LoginUITests.swift:42
  Likely cause (unverified): the SSO button's accessibility identifier changed

NEEDS REVIEW (spec flagged, not counted as a regression)
  TC-LOGIN-003  Login fails without internet: passed

SKIPPED
  TC-LOGIN-004  superseded by TC-LOGIN-018

NOT COVERED YET (spec exists, no test code)
  TC-LOGIN-009  Session restored after app relaunch

SUGGESTIONS (not applied)
  - TC-LOGIN-007: add accessibilityIdentifier "sso_google_button" to the SSO
    button in LoginView.swift so the test doesn't depend on the label text.
  - TC-LOGIN-002: spec says ui-single-screen; a unit test on LoginViewModel would
    cover the same behavior faster. Kept as ui-single-screen per spec.
  - Add .test-results/ to .gitignore.

Raw log: .test-results/2026-09-24-1432-smoke.log
```

Rules:

- The summary line counts passed, failed, and needs-review separately. A failing
  `needs-review` test case appears under NEEDS REVIEW, never under FAILED.
- Each failure names the spec step that failed, quoted from the spec's `Steps`, and
  shows expected versus actual.
- A cause is only stated as fact if the output proves it. Otherwise prefix it with
  "Likely cause (unverified)".
- Suggestions are listed, never applied. Level disagreements go here.
- Duration is taken from the tool output. Don't estimate it.

## Write mode

```
Wrote 3 tests for Login (iOS). Not run yet.

WRITTEN
  TC-LOGIN-001  unit              LoginTests.swift
  TC-LOGIN-002  ui-single-screen  LoginUITests.swift
  TC-LOGIN-003  integration       LoginTests.swift  (marked NEEDS-REVIEW)

ALREADY COVERED (left unchanged)
  TC-LOGIN-005  LoginTests.swift:88

WAITING ON YOUR ANSWER
  TC-LOGIN-007  Login via Google SSO
  The test would have to drive Google's sign-in sheet, which the app doesn't
  control. Mock at the auth service, or use a dedicated test account?

SUGGESTIONS (not applied)
  - ...

Run these now? TC-LOGIN-002 is a UI test and needs a simulator.
```

End write mode with the run question only when the user didn't already ask to run.
If the user asked for write then run, go straight to run mode after writing.
