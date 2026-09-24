# Changelog

Versions are tracked per plugin. Tags use the format `<plugin>-v<version>`.

## testrun-forgemaster 1.0.0

First release.

- Writes test code strictly from specs in `docs/test-cases/`, runs it, and
  reports results in plain language.
- Write, run, or both. Ambiguous requests default to write, then ask before
  running.
- Mandatory `Covers: TC-xxx` traceability comment on every generated test.
- Never modifies app code, never weakens assertions, never touches tests it
  didn't write. Fixes are suggested only.
- Confirmation before e2e or large UI runs. Zero executed tests is never
  reported as a pass.
- Backend runs refuse non-test databases or servers without confirmation.
- Run settings saved once to `.quality-atelier.json`.
- Platform packs: iOS, Android, Web, Backend/API.

## testcase-artisan 1.1.0

- Added Android, Web, and Backend/API platform packs.
- Backend packs can declare extra `Level test` values (`contract`) and mark
  UI-only levels as not applicable.
- `Level test` is now the reference that testrun-forgemaster follows, not a
  loose hint.
- Description now separates spec requests from test-code requests, and points
  to testrun-forgemaster for the latter.
- Stricter Android detection (`AndroidManifest.xml` required, so JVM backends
  aren't misdetected).
- Fixed wrong internal step references in SKILL.md.
- Removed an em dash and a non-English heading from the index format example.

## Repository

- Renamed from `testcase-artisan` to `quality-atelier` and restructured as a
  multi-plugin marketplace. Install with
  `/plugin marketplace add zhuhrymhd/quality-atelier`.

## testcase-artisan 1.0.0

First release (tag `v1.0.0`, before the repository was renamed).
