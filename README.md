# testcase-artisan

![Version](https://img.shields.io/badge/version-1.0.0-blue)
![License: MIT](https://img.shields.io/badge/license-MIT-green)
![Claude Code](https://img.shields.io/badge/Claude%20Code-plugin-orange)
![Platform: iOS](https://img.shields.io/badge/platform-iOS-lightgrey)

**Tags:** `testing` `test-cases` `qa` `specification` `edge-cases` `ios`

A Claude Code skill that generates and maintains structured test case
specifications from your project's documentation. It does not write or run
test code - it produces markdown test case specs that a separate testing
skill (or a human) can act on.

## What it does

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
- Ships an iOS platform pack today (detection signals for permissions,
  background/foreground, Dynamic Type, etc.); the core taxonomy is
  platform-neutral so other platform packs can be added later.

## Install

Pick **one** of the two options below.

**Option A - Claude Code's plugin marketplace**

Run these two commands one at a time in the Claude Code prompt:

1. Add the marketplace:
   ```
   /plugin marketplace add zhuhrymhd/testcase-artisan
   ```
2. Install the plugin:
   ```
   /plugin install testcase-artisan@testcase-artisan-marketplace
   ```

If you add the marketplace through the `/plugin` menu instead, enter only
`zhuhrymhd/testcase-artisan` as the marketplace source.

**Option B - skills.sh (works across Claude Code, Cursor, Codex, and others)**
```
npx skills add zhuhrymhd/testcase-artisan
```

## Usage

Invoke it explicitly with `/testcase-artisan`, or just describe what you
need in natural language, e.g. "generate test cases for the Login feature"
or "update test cases, I just added SSO to Login". Claude picks up the
skill automatically when relevant.

## Structure

```
testcase-artisan/
├── .claude-plugin/
│   ├── marketplace.json
│   └── plugin.json
└── skills/
    └── testcase-artisan/
        ├── SKILL.md
        └── references/
            ├── field-format.md
            ├── writing-style.md
            ├── edge-case-checklist.md
            ├── core-flows.md
            └── platforms/
                └── ios.md
```

## License

MIT (see LICENSE).
