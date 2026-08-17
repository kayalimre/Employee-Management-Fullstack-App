# Web UI Tester Profile

<!-- factory-version: 1.3.0 -->

> web-ui-tester tuning — Playwright web UI suite conventions. Read by the `web-ui-tester`.
> Last updated: 2026-08-17

## Suite Layout
- **No Playwright suite exists yet** — there is no `playwright.config.*`, no `@playwright/test` dependency, and no e2e directory in this repo. The existing frontend tests are Jest + Testing Library component tests, which are NOT browser UI tests.
- Spec directory (to create on first use): `webui/specs/`, helpers in `webui/helpers/`, config `playwright.config.ts` at the project root. Do not colonize `frontend/__tests__/` — that is the Jest suite.
- Naming: `<flow>.spec.ts` (e.g. `login.spec.ts`, `employee-crud.spec.ts`).

## Selector Policy
- Prefer `getByRole` / `getByLabel` / `getByText`; no positional or deep-CSS chains.
- Test-id attribute: **none in the application source** — a scan of `frontend/src/**` found no `data-testid` attributes; the existing Jest suite queries purely by role and label. Role/label selectors are therefore the primary strategy here. If a flow genuinely cannot be targeted accessibly, request a `data-testid` be added to the component (developer's job) rather than falling back to CSS/XPath.
- MUI renders semantic roles for buttons, dialogs, and text fields — lean on `getByRole('button', { name: ... })` and `getByLabel(...)`.
- The Three.js landing page (`components/hero3d/`) renders to a WebGL canvas with no accessible internals: assert on the surrounding DOM, never on canvas contents.

## Runtime
- App start for testing: the frontend needs the backend running. Two supported paths —
  - Docker (closest to production): `docker compose -f docker-compose.yml up --build` (frontend on `http://localhost:3000`, backend on `http://localhost:8080`); note the Makefile's `docker-up` target is broken (it references `docker-compose.yaml`).
  - Local dev: `npm run dev` at the root (concurrently runs `cd backend && mvn spring-boot:run` and `cd frontend && npm start`). Backend requires `MYSQL_*` and `JWT_SECRET` env vars (see `backend/example_config.properties`).
- baseURL: `http://localhost:3000` (CRA dev server / compose-mapped nginx). Confirm before a run; TBD for any deployed environment.
- Browser matrix: chromium (default). WebAuthn/passkey flows need Chromium's virtual authenticator via CDP — plan for `chromium` only on those specs.
- Run mode: headless by default; `--headed` / `headless: false` available for live-drive debugging.
- Auth setup: no seeded test account is documented. `config/DataInitializer.java` and `databases/sql/04_seed_data.sql` / `data.sql` are the places to look for pre-existing users; otherwise register a user through the UI in a setup step. Preferred approach — TBD for the team.

## Web UI Test Report JSON Schema (WEBUIREPORT-{slug}.json)

```json
{
  "test_plan": "TESTPLAN-{slug} | null",
  "mode": "author-and-run | run-only | author-only",
  "generated_by": "web-ui-tester",
  "command": "npx playwright test ...",
  "environment": { "playwright": "...", "node": "...", "browsers": ["chromium"], "headed": false, "base_url": "..." },
  "summary": { "total": 0, "passed": 0, "failed": 0, "flaky": 0, "skipped": 0,
               "pass_rate": "0%", "duration_ms": 0 },
  "specs": [ { "path": "webui/specs/....spec.ts", "status": "created | updated | unchanged",
               "covers": ["TC-1"] } ],
  "cases": [
    { "id": "TC-1", "spec": "webui/specs/....spec.ts", "traces_to": ["..."],
      "result": "pass | fail | blocked", "duration_ms": 0,
      "expected": "...", "actual": "...",
      "verdict": "product-wrong | test-wrong | env-wrong | null",
      "evidence": { "screenshot": "...", "trace": "..." }, "notes": "..." }
  ],
  "degraded": ["..."],
  "flaky_reruns": [ { "id": "TC-3", "first": "fail", "rerun": "pass" } ]
}
```
