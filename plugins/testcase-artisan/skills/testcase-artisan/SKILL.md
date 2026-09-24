---
name: testcase-artisan
description: >
  Creates and maintains structured test case specifications (not test code) for app
  features, derived from the project's documentation and specs. Use this whenever the
  user asks to write, generate, draft, or update test cases for a feature - including
  phrasing that doesn't name "test case" explicitly, like "what needs to be tested
  here", "what scenarios should this cover", "test cases for the new SSO flow", "is
  there anything we're missing for login", or wants QA coverage planned out right
  after finishing or describing a feature. Also use this to audit which features in
  the repo are missing test case coverage. This skill only produces test case
  specification documents in markdown. It does NOT write or execute test code, and it
  does NOT decide overall test strategy - that belongs to a separate testing skill.
---

# Testcase Artisan

Turns project documentation into structured, maintainable test case specifications.
Read this whole file before doing anything - the steps below depend on each other.

## Boundary

This skill produces one thing: markdown files describing what should be tested and
why, under `docs/test-cases/`. It never writes XCTest/XCUITest/Jest/etc. code, and it
never runs a build or a simulator. If the user asks to actually implement or run
tests, that's a different skill's job - point them there instead of doing it here.

The `Level test` field this skill writes on each test case is a classification, not a
final decision. A downstream testing skill may reasonably disagree with it once it
looks at real implementation constraints. Don't treat it as binding.

## Step 1: Figure out the mode

There is no special phrasing for "generate" vs "update" vs "audit" - infer the mode
from what's on disk and what the user asked for:

- User named a specific feature (or one is obvious from context) → **scoped mode**.
  Check whether `docs/test-cases/<feature>.md` already exists.
  - Doesn't exist → **generate**: build the file from scratch (Step 4 onward).
  - Exists → **update**: read it first, then reconcile against current docs (Step 6).
- User asked something like "what's missing", "audit coverage", or gave no specific
  feature at all → **gap-audit mode** (Step 7). Don't generate full test case detail
  in this mode - just report gaps.

Scoped mode is the default and should be cheap: only read documentation relevant to
the feature in question, not the whole repo. Gap-audit is the only mode that scans
broadly, and even then it only reports gaps - it doesn't fill them in.

## Step 2: Read the documentation, in priority order

For the feature in scope, look for sources in this order and prefer higher ones when
they conflict:

1. A dedicated spec folder (`docs/`, `specs/`, `PRD.md`, or similar)
2. A README scoped to that feature or module
3. Doc-comments in the code (`///` in Swift, JSDoc, docstrings, etc.)
4. The code's own structure, as a last resort when nothing above exists

Every test case must record which of these it came from. Use `Source: written spec`
when it's grounded in something actually written down, and `Source: inferred` when
you had to reason it out from code structure or from the edge-case checklist in Step
5. Never blend the two silently - the distinction tells the user how much to trust
the test case without re-reading the source themselves.

## Step 3: Detect the platform

Look for marker files at the repo root to decide which platform pack to load from
`references/platforms/`:

- `*.xcodeproj`, `Package.swift` → iOS (load `references/platforms/ios.md`)

Only `ios.md` exists today. If you detect a different stack and no matching platform
pack exists yet, say so explicitly to the user and fall back to the platform-neutral
categories in `references/edge-case-checklist.md` alone - don't invent platform
conventions that aren't documented.

In a monorepo with more than one platform, treat each platform's version of a
same-named feature as a separate output file (see Step 8 on ID prefixes) - don't
merge them into one document.

## Step 4: Build the test case set (generate mode)

For each thing the docs describe the feature doing, write one test case per distinct
behavior or condition, not one per docs paragraph. Then deliberately extend beyond
what's written:

1. Read `references/edge-case-checklist.md`. For every category listed, check its
   trigger condition against this feature. If the condition is met, add a test case
   for it, marked `Source: inferred (checklist: <category>)`.
2. Read `references/platforms/<platform>.md` for the detected platform and do the
   same for its platform-specific categories and detection signals.
3. Read `references/core-flows.md` to determine whether this feature counts as a core
   flow, and classify each test case's `Priority` accordingly (rules are in
   `references/field-format.md`).
4. Classify each test case's `Level test` from what its own Steps actually require -
   see `references/field-format.md` for the decision rule. Set `Smoke: true` only on
   test cases that are `P0` AND core-flow AND happy-path.
5. Write every test case following the exact field schema and writing style in
   `references/field-format.md` and `references/writing-style.md`. Don't skip the
   style guide - it's short and it's what keeps this skill's output distinct from
   generic prose.

Assign IDs sequentially starting at 001 within the feature (see Step 8 for the
prefix). Don't leave gaps and don't reuse numbers even across separate invocations.

## Step 5: Reconcile against existing test cases (update mode)

When `docs/test-cases/<feature>.md` already exists, never regenerate it from scratch
and never silently delete or rewrite an existing entry. Instead:

1. Read the existing file in full, including every test case's `Source` field.
2. Read the current documentation for the feature (Step 2).
3. For each existing test case, check whether the spec section its `Source` points to
   is still consistent with the current docs:
   - Still consistent, not touched by whatever changed → leave it as `active`.
   - The change plausibly affects it (even indirectly - e.g. a new auth method added
     alongside an existing one) → keep it, but set `Status: needs-review` and add a
     one-line reason. Do not edit its content to guess at the fix yourself.
   - The behavior it tested has been fully replaced by something else in the new
     test cases you're about to add → set `Status: superseded-by: <new-id>`, and
     still don't delete it.
4. For anything in the current docs that isn't covered by an existing test case
   (including a brand new sub-flow), generate new test cases following Step 4,
   continuing the ID sequence from the highest existing number in the file - never
   restart at 001.
5. Report a short summary of what changed: how many test cases were added, and which
   existing ones were flagged `needs-review` or `superseded-by`, with the reason.

## Step 6: Check for auto-split

After writing or updating a feature file, check whether it now has either more than
15 test cases in one section, or 2 or more clearly named sub-flows (like the SSO
example: `Login` containing both a base flow and a distinct `SSO` sub-flow).

If either is true, split the sub-flow out into its own file (e.g. `login-sso.md`),
leave a one-line cross-reference in both files pointing to the other, and tell the
user explicitly that you did this and why - don't restructure files silently.

## Step 7: Gap-audit mode

When the user wants a repo-wide check instead of one feature:

1. Find all features that have documentation under the sources in Step 2.
2. For each, check whether `docs/test-cases/<feature>.md` exists.
3. Report the list of features with no test case file at all - this is the entire
   output of this mode. Don't generate test case detail for the gaps found; that
   happens later as a normal scoped invocation once the user picks one.

## Step 8: Write the output files

- Main file: `docs/test-cases/<feature>.md` (or `<feature>-<subflow>.md` after a
  split). This is the source of truth - full field detail, per `field-format.md`.
- `docs/test-cases/_index.md`: always regenerate this in full from the current state
  of every feature file after any change. It's a derived summary, not something a
  human edits by hand, so overwriting it completely is fine (unlike the feature
  files themselves). One checklist line per test case: ID, priority, level test,
  smoke flag if set, and a short title.
- ID format: `TC-<FEATURE>-<NUMBER>` normally. In a monorepo with more than one
  platform, prefix with the platform too: `TC-IOS-LOGIN-003`, so the same feature
  name on two platforms never collides.

## Reference files

Load these as needed rather than assuming their contents:

- `references/field-format.md` - exact field schema, ID rules, and the decision
  rules for Priority and Level test.
- `references/writing-style.md` - the writing rules every field value must follow.
- `references/edge-case-checklist.md` - the platform-neutral categories and their
  trigger conditions, used in Step 4.
- `references/core-flows.md` - where the user maintains their explicit list of core
  flows, and the fallback rule for features not on it.
- `references/platforms/ios.md` - iOS detection signals and iOS-specific categories.
