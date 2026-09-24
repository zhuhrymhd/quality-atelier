# Platform Pack: Backend / API

## Required config

`runner`, `baseUrl` (only if tests call a running server), and
`testDatabaseUrlEnv` (the name of the environment variable holding the test
database URL, never the URL itself) if tests touch a database.

## Safety rule: never touch a non-test environment

Before running any `integration` or `contract` test:

1. Resolve the database URL from the environment variable named in
   `testDatabaseUrlEnv`, and the `baseUrl`.
2. If either points to anything other than `localhost`, `127.0.0.1`, a Docker
   service name, or a host or database name that clearly contains `test`, stop.
   Show the user the host (not the full URL or credentials) and ask for explicit
   confirmation.
3. If the environment variable is unset, stop and say so. Never fall back to the
   app's default database settings; they often point to a development or
   production database.

Tests may create, change, and delete data. That's why this check is not optional.

## Frameworks

Match the stack and whatever the project already uses.

| Stack | `unit` / `integration` | Run a filtered test |
|---|---|---|
| Node.js | Jest or Vitest, with supertest for HTTP | `npx vitest run tests/login.test.ts -t "TC-API-LOGIN-004"` |
| Python | pytest (FastAPI `TestClient`, Django test client, or httpx) | `pytest tests/test_login.py -k "tc_api_login_004"` |
| Go | `testing` package with `net/http/httptest` | `go test ./internal/auth -run 'TestTCAPILogin004'` |
| Java / Kotlin (Spring) | JUnit 5 with MockMvc or WebTestClient | `./gradlew test --tests "*LoginApiTest*"` or `./mvnw test -Dtest=LoginApiTest#tcApiLogin004` |
| Ruby | RSpec or Minitest | `bundle exec rspec spec/requests/login_spec.rb -e "TC-API-LOGIN-004"` |

Put the test case ID in the test name, adapted to the language's naming rules, so
the filter can select it. The `Covers:` comment (`#` in Python and Ruby, `//`
elsewhere) is still required.

`ui-single-screen` and `e2e-multi-screen` don't apply on this platform. If a
backend spec uses them, report it as a spec problem and suggest correcting it with
testcase-artisan; don't implement it.

## `contract` level

Validate responses against the project's own schema (OpenAPI/Swagger, GraphQL
schema, protobuf). Use what the project already has: a schema-validation library,
Schemathesis for OpenAPI in Python, or the framework's built-in response
validation. If nothing is available, ask before adding a tool.

A contract test asserts shape (status code, required fields, types, enum values),
not business behavior. Behavior belongs to `integration`.

## Files

Follow the stack's convention and the project's existing layout: `tests/` for
pytest, `_test.go` next to the code (same package) for Go, `src/test/` for
Gradle/Maven, `spec/` for RSpec, next to the source or in `tests/` for Node.

## Backend-specific uncertainty triggers

- The spec requires an external service (payment provider, email, third-party API)
  and the project has no existing mock or sandbox for it.
- Rate limiting or timeout tests that would need real waiting time longer than a
  few seconds, or a clock the project doesn't let tests control.
- Concurrency or race-condition tests where the spec doesn't define how many
  concurrent requests or what outcome counts as correct.
- A migration test that would run migrations against a database the safety rule
  can't confirm is a test database.
