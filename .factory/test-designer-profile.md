# Test Designer Profile

<!-- factory-version: 1.3.0 -->

> test-designer tuning + the test-plan template and JSON schema. Read by the `test-designer`.
> Last updated: 2026-08-17

## Test Plan Template
- Standard: IEEE 829 (default). No custom project test-plan template exists.

## Scope & Coverage
- Integration tests, traced to plan acceptance criteria. Coverage expectation: every P0/P1 acceptance criterion — TBD for the team to confirm a stricter bar.
- The system is a REST backend (`http://localhost:8080`) plus a React SPA (`http://localhost:3000`); most plans will mix cURL cases against the API with Playwright cases against the UI.
- Setup guidance to reuse in plans: `docker compose -f docker-compose.yml up --build` brings up backend + frontend + MySQL 8 + MongoDB 6 (note: the Makefile `docker-up` target is broken — it references `docker-compose.yaml`, the file is `docker-compose.yml`). Local alternative: root `npm run dev`, with `MYSQL_*` and `JWT_SECRET` env vars set per `backend/example_config.properties`. Health endpoint: `GET /api/health`.
- Teardown guidance: `docker compose -f docker-compose.yml down`; when reusing a live DB, plans must clean up records they created (Hibernate runs `ddl-auto=update`, so schema is not reset between runs).

## Synthetic data defaults
- Default all synthetic PII/fixtures to the reserved, non-routable ranges in `claude-plugin/references/safe-synthetic-data.md` (RFC 2606 domains `@example.com`, reserved phone blocks `555-01NN`, RFC 5737 IPs `192.0.2.0/24`, standard test cards `4242…`, obviously-fictional names/IDs) — never a real corporate domain, live phone, or routable IP.
- Project-specific fixtures/allowed real values: TBD. Existing seed data lives in `data.sql`, `databases/sql/04_seed_data.sql`, `databases/mongo/03_seed_data.js`, and `config/DataInitializer.java` — check there before inventing accounts, and treat any credentials found there as local-dev-only.

## Test Technique (by platform, from project-profile)
- Backend/API → cURL against `http://localhost:8080` (endpoints documented in root `openapi.yaml` and springdoc Swagger UI). JWT bearer token obtained from the auth endpoint in setup.
- Web → Playwright CLI against `http://localhost:3000`. No Playwright suite exists yet, so web cases mark the `web-ui-tester` as the authoring owner.
- Mobile → not applicable (no mobile app in this project).
- WebAuthn/passkey flows cannot be exercised by plain cURL or ordinary browser automation — they need Chromium's virtual authenticator (Playwright + CDP). Design those cases explicitly as `playwright-cli`, and state the virtual-authenticator setup in the case's `setup`.

## Error-code coverage defaults
- Per testable API operation, walk the canonical error-code matrix (`claude-plugin/references/error-code-matrix.md`) and design a negative/boundary case per **applicable** code — or an N/A stub (`reason: "not-applicable"`) when it can't arise. Applicability, not blanket generation.
- In-scope-by-default codes for this project: `400` (Bean Validation failures on request DTOs), `401` (missing/expired/invalid JWT), `403` (authenticated but not permitted), `404` (unknown employee/department id), `409` (duplicate username/email/department), `422` where the API uses it, `500` (unhandled server error). `429` is **out of scope by default** — no rate limiting is implemented; include it only once one exists. `502/503`/timeout apply to the compose/k8s deployment paths, not to a local single-process run.
- Hygiene cases generated **above** the FRD's explicit acceptance criteria are tagged `hygiene` / `above-FRD`.

## Test Plan JSON Schema (canonical output for TESTPLAN-{slug}.json)

```json
{
  "requirement": "REQ-{slug}",
  "plan": "PLAN-{slug}",
  "generated_by": "test-designer",
  "test_plan": {
    "identifier": "TESTPLAN-{slug}",
    "introduction": "...",
    "test_items": ["..."],
    "features_to_test": ["..."],
    "features_not_to_test": ["..."],
    "approach": "...",
    "pass_fail_criteria": "...",
    "suspension_resumption": "...",
    "environmental_needs": "...",
    "risks": ["..."]
  },
  "test_cases": [
    {
      "id": "TC-1",
      "traces_to": ["TASK-1.1.1#AC1", "TASK-1.1.1#AC2"],
      "title": "...",
      "setup": "preconditions + how to get the software running",
      "technique": "curl | playwright-cli | accessibility | integration",
      "steps": ["..."],
      "test_data": "...",
      "expected_result": "...",
      "teardown": "..."
    }
  ],
  "excluded_coverage": [
    { "excluded": "<scenario/behavior deliberately NOT covered here>", "reason": "already-covered-by-existing-asset", "covered_by": "<verifiable pointer: TC id / spec / suite / file:path>" }
  ],
  "traceability": {
    "_computed": "Written by /factory:plan step 5b via servers/lib/plan-verify.mjs — NOT hand-authored.",
    "test_case_total": 0,
    "tasks": { "total": 0, "covered": [], "uncovered": [] },
    "acceptance_criteria": { "total": 0, "traced": 0, "untraced": [], "unverified": 0, "granularity": "per-ac | task-level", "verdict": "complete | partial | unverified" }
  }
}
```

**`traces_to` references acceptance criteria by ID:** `TASK-KEY#ACn` = criterion *n* (1-based) of that task's `acceptance_criteria[]`; a bare `TASK-KEY` is task-level only (cannot certify per-AC coverage). The **`traceability`** block is **computed** from the test cases' `traces_to` against the plan's task ACs by the `/factory:plan` flow — never self-reported. `verdict` is `complete` only when every task AC is AC-traced and covered; `partial` when an AC has no test; `unverified` when a task is traced only at task level. See `references/plan-schema.md`.
