# Platform Pack: Android

## Required config

`module` (usually `app`), `variant` (usually `debug`), and `avd` for any run that
includes instrumented tests.

Detect options before asking:

```
./gradlew projects          # modules
emulator -list-avds         # available emulators
adb devices                 # running emulators or connected devices
```

## Frameworks

Use what the project already uses. Check `build.gradle(.kts)` dependencies and
existing tests first.

| Level test | Framework | Where it lives |
|---|---|---|
| `unit` | JUnit 4 or 5, matching existing tests | `<module>/src/test/` |
| `integration` | JUnit with fakes or mocks (MockK, Mockito) injected through the app's existing seams | `<module>/src/test/` |
| `snapshot` | Paparazzi or Roborazzi, only if already a dependency | As that library expects |
| `ui-single-screen` | Compose UI test (`createComposeRule`) for Compose screens, Espresso for View-based screens | `<module>/src/androidTest/` (or `src/test/` with Robolectric, if the project already does that) |
| `e2e-multi-screen` | Espresso or Compose UI test driving the real Activity | `<module>/src/androidTest/` |

Gradle only picks up tests from these source sets. Don't place test files anywhere
else. If a needed source set or library is missing, ask; don't edit
`build.gradle(.kts)`.

## Files

- One file per feature per source set: `LoginTest.kt` in `src/test/`,
  `LoginUiTest.kt` in `src/androidTest/`, in the same package as the code under
  test. Follow the project if it organizes tests differently.

## Run commands

Local (JVM) tests, no emulator needed:

```
./gradlew :app:testDebugUnitTest --tests "com.example.login.LoginTest"
```

Instrumented tests, need a running emulator or device:

```
./gradlew :app:connectedDebugAndroidTest \
  -Pandroid.testInstrumentationRunnerArguments.class=com.example.login.LoginUiTest#loginViaGoogleSso
```

- Build the task name from config: `test<Variant>UnitTest` and
  `connected<Variant>AndroidTest`, with the variant capitalized.
- Check `adb devices` first. If no device is running, starting the configured AVD
  (`emulator -avd <avd>`) is part of the run, which is why Step 6 of SKILL.md
  requires confirmation for UI levels.
- If a filtered run reports zero tests executed, report the filter as wrong. Don't
  call it a pass.

## Writing conventions

- Find Compose elements by `testTag`, View elements by resource ID. If the element
  has neither, that's an uncertainty trigger: ask, and suggest the tag or ID in the
  report.
- Never use `Thread.sleep`. Use Compose `waitUntil`, or an Espresso
  `IdlingResource` if the project already has one.
- Runtime permission dialogs are system UI. Use `GrantPermissionRule` when the
  spec's precondition says the permission is granted. When the spec tests the denial
  path, it needs UI Automator to tap the system dialog; if UI Automator isn't a
  dependency, ask.

## Android-specific uncertainty triggers

- Process death or configuration-change tests where the spec doesn't say which one
  (they need different techniques: `ActivityScenario.recreate()` versus killing the
  process with `adb shell am kill`).
- Deep links or intents from outside the app where the spec doesn't define the URI
  or extras.
- OEM-specific behavior (battery optimization, custom permission screens) that the
  configured emulator can't reproduce. Say so instead of writing a test that can't
  fail.
