# Platform Pack: Web

## Required config

`unitRunner` (`vitest` or `jest`), `e2eRunner` (`playwright` or `cypress`),
`baseUrl`, and `browser` for e2e runs.

Detect options before asking: check `package.json` dependencies and scripts, and
look for `vitest.config.*`, `jest.config.*`, `playwright.config.*`,
`cypress.config.*`. If `playwright.config.*` defines `webServer` and `use.baseURL`,
take `baseUrl` from there instead of asking.

## Frameworks

| Level test | Framework | Where it lives |
|---|---|---|
| `unit` | Vitest or Jest, matching the project | Next to the source file (`*.test.ts`) or in `__tests__/`, matching existing tests |
| `integration` | Vitest or Jest, with network mocked through the project's existing approach (MSW, fetch mock, etc.) | Same as unit |
| `snapshot` | Playwright `toHaveScreenshot()` (a visual comparison). Jest/Vitest DOM snapshots are not visual; don't use them for this level | Playwright test directory |
| `ui-single-screen` | Testing Library (`@testing-library/*`) component test if the project uses it; otherwise Playwright against one route | Unit test location, or Playwright test directory |
| `e2e-multi-screen` | Playwright (or Cypress if that's what the project uses) | The runner's configured test directory (`testDir` in `playwright.config.*`) |

If a runner or library the level needs isn't installed, ask. Don't add it to
`package.json`.

## Files

- One file per feature per level: `login.test.ts` for unit/integration,
  `login.spec.ts` in the e2e directory. Match the project's naming and extension
  (`.js`, `.ts`, `.tsx`).

## Run commands

Use the project's package manager (check for `pnpm-lock.yaml`, `yarn.lock`,
`bun.lockb`, or `package-lock.json`). Examples with npm:

```
npx vitest run src/features/login/login.test.ts -t "TC-LOGIN-002"
npx jest src/features/login/login.test.ts -t "TC-LOGIN-002"
npx playwright test e2e/login.spec.ts --project=chromium -g "TC-LOGIN-007"
```

- Include the test case ID in every test name (`test("TC-LOGIN-007 login via Google
  SSO", ...)`) so `-t` / `-g` filters can select it. The `// Covers:` comment is
  still required above it.
- Playwright needs the app running at `baseUrl`. If `webServer` is configured,
  Playwright starts it. If not, check whether `baseUrl` responds before running; if
  it doesn't, ask the user to start the app. Don't start long-running dev servers
  yourself without asking.
- If a filtered run reports zero tests executed, report the filter as wrong. Don't
  call it a pass.

## Writing conventions

- Select elements by accessible role and name (`getByRole("button", { name:
  "Log in" })`), then label, then `data-testid`. If none of these is stable, that's
  an uncertainty trigger.
- Never use fixed waits (`waitForTimeout`, `setTimeout`). Use web-first assertions
  (`await expect(locator).toBeVisible()`).
- Browser permission prompts: use Playwright's `context.grantPermissions` for the
  granted path. The denied path is the default state in a fresh context.

## Web-specific uncertainty triggers

- The spec requires a browser other than the configured one, or cross-browser
  coverage, and those browsers aren't installed (`npx playwright install` would
  download them; ask first).
- The flow redirects to a third-party domain (OAuth provider, payment page).
- Multi-tab behavior where the spec doesn't say how the second tab is opened.
