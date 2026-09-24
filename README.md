# quality-atelier

![License: MIT](https://img.shields.io/badge/license-MIT-green)
![Claude Code](https://img.shields.io/badge/Claude%20Code-plugin%20marketplace-orange)
![Platforms](https://img.shields.io/badge/platforms-iOS%20%7C%20Android%20%7C%20Web%20%7C%20Backend-lightgrey)

**Tags:** `testing` `test-cases` `qa` `specification` `test-automation` `ios` `android` `web` `backend`

A Claude Code plugin marketplace for QA work. It ships two plugins that work
as a pair: one writes the test case specs, the other turns those specs into
test code and runs it.

| Plugin | Version | What it does |
|---|---|---|
| `testcase-artisan` | 1.1.0 | Writes and maintains test case specs (markdown) from your project docs. Does not write or run test code. |
| `testrun-forgemaster` | 1.0.0 | Writes and runs test code strictly from those specs, then reports results in plain language. Never modifies app code. |

## testcase-artisan

- Reads your docs/specs in a defined priority order and turns them into
  structured test cases, one file per feature under `docs/test-cases/`.
- Extends beyond what's explicitly written, using a trigger-condition-based
  edge case checklist (network failure, permission denied, empty state,
  session expiry, and more) instead of guessing freely.
- Updates safely when a feature changes: existing test cases are never
  silently deleted or overwritten. Ones that may be affected are flagged
  `needs-review` with a reason; superseded ones are flagged, not removed.
- Classifies each test case by priority (P0/P1/P2) and test level
  (unit / integration / snapshot / ui-single-screen / e2e-multi-screen)
  using explicit, repeatable rules.
- Platform packs: iOS, Android, Web, Backend/API.

## testrun-forgemaster

- Writes test code strictly from specs in `docs/test-cases/`, runs it, and
  reports results in plain language.
- Every generated test carries a `Covers: TC-xxx` comment that traces it back
  to its spec.
- Never modifies app code, never weakens assertions, and never touches tests
  it didn't write. Fixes are suggested only.
- Asks before e2e or large UI runs. Zero executed tests is never reported as
  a pass.
- Run settings are saved once to `.quality-atelier.json`.
- Platform packs: iOS, Android, Web, Backend/API.

If no spec exists for a feature yet, testrun-forgemaster tells you to run
testcase-artisan first.

## Install

Pick **one** of the two options below.

**Option A - Claude Code's plugin marketplace**

Run these commands one at a time in the Claude Code prompt:

1. Add the marketplace:
   ```
   /plugin marketplace add zhuhrymhd/quality-atelier
   ```
2. Install the plugins you want:
   ```
   /plugin install testcase-artisan@quality-atelier
   /plugin install testrun-forgemaster@quality-atelier
   ```

If you add the marketplace through the `/plugin` menu instead, enter only
`zhuhrymhd/quality-atelier` as the marketplace source.

**Option B - skills.sh (works across Claude Code, Cursor, Codex, and others)**
```
npx skills add zhuhrymhd/quality-atelier
```

## Usage

- Specs: `/testcase-artisan`, or ask in plain language, e.g. "generate test
  cases for the Login feature" or "update test cases, I just added SSO to
  Login".
- Test code: `/testrun-forgemaster`, or ask e.g. "write the UI test for SSO"
  or "run the smoke tests".

Claude also picks up the right skill automatically when relevant.

## Structure

```
quality-atelier/
├── .claude-plugin/
│   └── marketplace.json
└── plugins/
    ├── testcase-artisan/
    │   ├── .claude-plugin/plugin.json
    │   └── skills/testcase-artisan/
    │       ├── SKILL.md
    │       └── references/
    │           ├── core-flows.md
    │           ├── edge-case-checklist.md
    │           ├── field-format.md
    │           ├── writing-style.md
    │           └── platforms/ (ios, android, web, backend-api)
    └── testrun-forgemaster/
        ├── .claude-plugin/plugin.json
        └── skills/testrun-forgemaster/
            ├── SKILL.md
            └── references/
                ├── config.md
                ├── field-format.md
                ├── report-format.md
                ├── writing-style.md
                └── platforms/ (ios, android, web, backend-api)
```

## Changelog

See [CHANGELOG.md](CHANGELOG.md).

## License

MIT (see LICENSE).
