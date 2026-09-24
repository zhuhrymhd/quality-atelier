# Platform Pack: iOS

## Required config

`scheme`, `simulator`, `os`, and either `workspace` or `project`.

Detect options before asking:

```
xcodebuild -list -workspace MyApp.xcworkspace    # schemes (or -project MyApp.xcodeproj)
xcrun simctl list devices available              # simulators and OS versions
```

## Frameworks

Use what the project already uses. Check existing test files first.

| Level test | Framework | Where it lives |
|---|---|---|
| `unit` | XCTest (`XCTestCase`) or Swift Testing (`import Testing`, `@Test`), matching existing tests | Unit test target (usually `<App>Tests`) |
| `integration` | Same as unit, with mocked network/storage injected through the app's existing seams | Unit test target |
| `snapshot` | swift-snapshot-testing (`assertSnapshot`) only if the package is already a dependency | Unit test target |
| `ui-single-screen` | XCUITest (`XCUIApplication`) | UI test target (usually `<App>UITests`) |
| `e2e-multi-screen` | XCUITest | UI test target |

If the project has no tests at all, prefer Swift Testing for unit and integration
on Xcode 16 or later, and say so in the report. If snapshot testing is needed and
the package isn't present, ask before continuing; don't add it.

## Files

- One file per feature per target: `LoginTests.swift` in the unit target,
  `LoginUITests.swift` in the UI test target. If the project groups tests
  differently, follow the project.
- Target membership: if `project.pbxproj` contains `PBXFileSystemSynchronizedRootGroup`
  for the test target's folder (Xcode 16 synchronized folders), a new file placed in
  that folder is included automatically. Otherwise, a new file must be added to the
  target in Xcode. Don't edit `project.pbxproj`; tell the user to add the file to
  the target, and name the file and the target. Adding test functions to an
  existing file needs nothing extra.
- If the unit or UI test target doesn't exist, ask the user to create it in Xcode
  (File > New > Target). Don't create it.

## Run commands

```
xcodebuild test \
  -workspace MyApp.xcworkspace -scheme MyApp \
  -destination 'platform=iOS Simulator,name=iPhone 16 Pro,OS=18.0' \
  -only-testing:MyAppUITests/LoginUITests/testLoginViaGoogleSSO \
  -resultBundlePath .test-results/2026-09-24-1432-login.xcresult
```

- Use `-project` instead of `-workspace` when there's no workspace.
- Repeat `-only-testing:` once per selected test. Format:
  `<Target>/<Class>/<method>`. Swift Testing identifiers can differ (for example, a
  trailing `()` on the function name). If a filtered run reports zero tests
  executed, treat the filter as wrong: report it instead of claiming a pass.
- Unit-only runs can skip the UI test target entirely. Mixed selections run in one
  command.
- A UI run includes building and booting the simulator. That's why Step 6 of
  SKILL.md requires confirmation.

## Writing conventions

- Find elements by `accessibilityIdentifier`. If the element has none, that's an
  uncertainty trigger: ask, and suggest the identifier in the report. Don't fall
  back to matching visible label text silently; if the user agrees to label text,
  note that the test will break on copy or localization changes.
- Wait with `waitForExistence(timeout:)`. Never use `sleep`.
- Pass setup state through `app.launchArguments` or `app.launchEnvironment` only if
  the app already reads them. If the test needs a mode the app doesn't support (for
  example, a mocked backend), suggest adding it; don't add it.
- System permission alerts belong to Springboard. Handle them with
  `addUIInterruptionMonitor`, or reset with
  `xcrun simctl privacy <device> reset all <bundle-id>` before a run when the spec's
  precondition requires a fresh permission state.

## iOS-specific uncertainty triggers

- Sign in with Apple or any `ASWebAuthenticationSession` flow (runs out of process,
  not reliably driveable from XCUITest).
- Push notification delivery, which needs `xcrun simctl push` with a payload the
  spec doesn't define.
- Widgets or Live Activities (outside the app's UI hierarchy).
