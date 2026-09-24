# Configuration: `.quality-atelier.json`

One file at the repo root, shared by every plugin in the quality-atelier suite. Each
plugin owns one top-level key and never writes to another plugin's key.

```json
{
  "testrun-forgemaster": {
    "ios": {
      "workspace": "MyApp.xcworkspace",
      "scheme": "MyApp",
      "simulator": "iPhone 16 Pro",
      "os": "18.0"
    },
    "android": {
      "module": "app",
      "variant": "debug",
      "avd": "Pixel_8_API_35"
    },
    "web": {
      "unitRunner": "vitest",
      "e2eRunner": "playwright",
      "baseUrl": "http://localhost:5173",
      "browser": "chromium"
    },
    "backend": {
      "runner": "pytest",
      "testDatabaseUrlEnv": "TEST_DATABASE_URL",
      "baseUrl": "http://localhost:8000"
    }
  }
}
```

Only the section for the detected platform is required. The exact required keys per
platform are listed in each platform pack.

## First run

1. Check whether the file exists and has a section for the detected platform.
2. If not, detect whatever you can first (list schemes, simulators, AVDs, runners
   using the platform pack's commands), then ask the user to confirm or choose, in
   one message. Offer detected options instead of asking open questions.
3. Save the answers. If the file exists, add only this plugin's section and leave
   the rest of the file untouched.
4. Tell the user: "Saved to .quality-atelier.json. Edit this file to change the
   simulator, scheme, or other run settings."

## Rules

- Never store secrets. For credentials, test account passwords, or database URLs,
  store the name of an environment variable (as in `testDatabaseUrlEnv`), never the
  value.
- If a saved value no longer works (the simulator was deleted, the scheme was
  renamed), stop and tell the user which value failed and how to fix it in the file.
  Don't silently pick another one.
