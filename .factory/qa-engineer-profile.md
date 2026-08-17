# QA Engineer Profile

<!-- factory-version: 1.3.0 -->

> qa-engineer tuning. Read by the `qa-engineer`.
> Last updated: 2026-08-17

## Execution Environment
- Bringing the system up (beyond what each plan states):
  - Containers: `docker compose -f docker-compose.yml up --build` — backend `:8080`, frontend `:3000` (nginx), MySQL 8 `:3306`, MongoDB 6 `:27017`. Do **not** use `make docker-up`; that target points at a non-existent `docker-compose.yaml`.
  - Local processes: root `npm run dev` (backend `mvn spring-boot:run` + frontend `npm start`).
  - Backend-only: `cd backend && mvn spring-boot:run`.
- Required backend env vars: `MYSQL_HOST`, `MYSQL_PORT`, `MYSQL_DB`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_SSL_MODE`, `JWT_SECRET`; optional `MONGO_URI`, `WEBAUTHN_RP_ID`, `WEBAUTHN_RP_NAME`, `WEBAUTHN_ALLOWED_ORIGINS`, `WEBAUTHN_CEREMONY_TIMEOUT_SECONDS`, `WEBAUTHN_ALLOW_ORIGIN_PORT`. Template: `backend/example_config.properties` → copy to the untracked `backend/config.properties`.
- Readiness probe before executing cases: `curl -f http://localhost:8080/api/health` (the compose healthcheck uses the same endpoint); frontend readiness: `curl -f http://localhost:3000`.
- API surface reference: root `openapi.yaml` and the springdoc Swagger UI on the running backend.
- Tooling: cURL for API cases; Playwright CLI for web cases (**not installed in this repo** — no `@playwright/test` dependency and no config; a web run requires the `web-ui-tester` to establish the suite first, otherwise report it as a structured net-blocker rather than improvising browser automation). No mobile targets.
- Seed/state: Hibernate runs `ddl-auto=update` and does not reset between runs; `config/DataInitializer.java`, `data.sql`, and `databases/sql/04_seed_data.sql` are the seed sources. Clean up records created during a run.
- **Assertion-timeout re-probe** — on a visibility/presence assertion timeout, do **one** markedly-longer re-probe before any regression verdict (never a loop). Extended budget = default assertion budget × **multiplier** `×5` (TBD to tune), capped at **ceiling** `30000 ms` (TBD to tune). Element appears within the extended budget → `slow-render` (records T1 budget + T2 appeared; not a FAIL); still absent → FAIL (genuine DOM/selector regression). Relevant here: the CRA dev server's first compile and the Spring Boot cold start are both slow, and `frontend/src/utils/warmup.js` exists specifically to mitigate a hosted cold-start delay.

## Evidence & Reporting
- Evidence to capture per case: full cURL request/response (status, headers, body), Spring Boot console/container logs for failures (`docker compose logs backend`), browser screenshot + Playwright trace for web cases, and the surefire XML (`backend/target/surefire-reports/`) when a plan invokes backend tests.
- **PII/secret masking (on by default).** Captured evidence (DOM/accessibility snapshots, response bodies, logs) can embed **real** personal data when the app under test renders it. Mask sensitive spans **as evidence is transcribed** into the report — detect by pattern (email/phone/Luhn-valid card/national-id/token shapes) **and** by label/accessibility-name context (`password`/`token`/`authorization`/`email`/`user`/`created-by`/`ssn`/`phone`/`card`). Two tiers: secrets → fixed-width placeholder (content **and** length hidden); identifying PII → partial deterministic mask keeping a small anchor (`mi•••@ad•••.com.tr`, `+90 ••• ••• •• 67`, `•••• •••• •••• 4242`). Unclassifiable → full mask (fail-safe). Structural fields (ids, `traces_to`, selectors, expected results) are never masked.
- Project-specific masking targets: JWT bearer tokens in `Authorization` headers and login responses, WebAuthn credential ids / public keys / challenges, employee email and phone fields, and the `JWT_SECRET` / `MYSQL_PASSWORD` values if they ever surface in a log line. Environment: `docker-compose.yml` credentials are known dev defaults, but still mask them in reports.
- Project overrides (extra sensitive labels, or masking off for a fully-synthetic environment): TBD.

## QA Report JSON Schema (QAREPORT-{slug}.json)

```json
{
  "test_plan": "TESTPLAN-{slug}",
  "mode": "inline | regression",
  "generated_by": "qa-engineer",
  "summary": { "total": 0, "passed": 0, "failed": 0, "blocked": 0, "slow_render": 0, "pass_rate": "0%" },
  "cases": [
    { "id": "TC-1", "traces_to": ["..."], "result": "pass | fail | blocked | slow-render",
      "failed_phase": "setup | run | null", "blocked_by": "<precondition-signature> | null",
      "reprobe": { "budget_ms": 0, "extended_ms": 0, "appeared_at_ms": 0 },
      "expected": "...", "actual": "...", "evidence": "...", "notes": "..." }
  ],
  "precondition_blocks": [
    { "precondition": "<failed shared setup step>", "affected_cases": ["TC-2", "TC-3"], "classification": "precondition-blocked",
      "kind": "env | auth | network | seed | null", "missing": "<the specific unmet prerequisite, named — never a guessed value>", "remediation": "<how the operator unblocks it>" }
  ]
}
```

A **structured net-blocker** is a `precondition_blocks[]` entry with `kind` set to `env`/`auth`/`network`/`seed`: emitted when a run can't proceed because an environment/auth/network/seed prerequisite is unmet, instead of a generic suspend. `missing` **names** the prerequisite (e.g. "env var `JWT_SECRET`", "a reachable MySQL instance on `MYSQL_HOST:MYSQL_PORT`", "an installed Playwright suite for web cases") read from the test plan's `environmental_needs`/case setup + visible config — it **never fabricates or guesses a credential value**; `remediation` says how to provide it and `affected_cases` lists the cases it unblocks. `kind: null` (or omitted) is an ordinary shared-setup precondition-block.
