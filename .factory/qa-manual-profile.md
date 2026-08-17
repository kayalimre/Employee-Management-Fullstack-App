# QA Manual Profile

<!-- factory-version: 1.3.0 -->

> qa-manual agent tuning. Read by the `qa-manual` agent.
> Last updated: 2026-08-17

## Test categories
- Authentication: username/password sign-in and registration, JWT session behavior (expiry, logout), protected-route redirects.
- Passkeys / WebAuthn: register a passkey, sign in with a passkey, list/remove passkeys, unsupported-device fallback, ceremony timeout (default 300s), origin/rp-id mismatch behavior.
- Account recovery: username verification → password reset.
- Employee management: list, search/sort as offered, create, edit, delete, validation errors, not-found handling.
- Department management: list, create (including the separate "new department" flow), edit, delete, and employee↔department association effects.
- Dashboard & reporting: chart rendering with data, with no data, and after a record changes; quick actions.
- Profile: view and update the signed-in user's details.
- Navigation & shell: navbar/footer, landing page (including the Three.js hero on low-end/WebGL-disabled clients), 404 page, loading overlay.
- Cross-cutting: form validation messages, toast error/success notifications, responsive layout (mobile horizontal-overflow has regressed before), browser back/forward, page refresh preserving session, slow/cold-start first load.
- API-level: the endpoints documented in root `openapi.yaml` / Swagger UI, exercised for both success and the error codes in `.factory/qa-gaps-profile.md`.

## Priority scheme
- **P0** — authentication (password and passkey), authorization/protected routes, and employee/department create+read: a failure blocks core use or exposes data.
- **P1** — update/delete flows, password reset, profile, dashboard data correctness, validation and error messaging.
- **P2** — cosmetic/visual, landing-page 3D effects, chart styling, non-blocking responsive polish.

## Scope
- Default: full application sweep. Feature-scoped and PR-scoped runs are both supported — for a PR-scoped run, derive the scope from the changed paths (`backend/**` → API + affected flows, `frontend/src/components/**` → the corresponding screens).
- Environments: local (`npm run dev`) or containers (`docker compose -f docker-compose.yml up --build`; the Makefile's `docker-up` target is broken). Frontend `:3000`, backend `:8080`, health `GET /api/health`.

## Synthetic data defaults
- All test accounts, employee records, emails, phone numbers, and addresses must use the reserved, non-routable ranges in `claude-plugin/references/safe-synthetic-data.md`: RFC 2606 domains (`@example.com`), reserved phone blocks (`555-01NN`), RFC 5737 IPs (`192.0.2.0/24`), standard test card numbers (`4242…`), and obviously fictional names/IDs.
- Existing seed data lives in `data.sql`, `databases/sql/04_seed_data.sql`, `databases/mongo/03_seed_data.js`, and `config/DataInitializer.java` — check there before inventing accounts; treat anything found there as local-development-only and never publish credential values in a scenario.
- Note that Hibernate runs with `ddl-auto=update` and the database is not reset between runs, so scenarios must state their own cleanup.
- Project overrides / allowed real values: TBD.
