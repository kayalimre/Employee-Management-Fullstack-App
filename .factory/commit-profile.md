# Commit Profile

<!-- factory-version: 1.3.0 -->

> commit-agent tuning.
> Last updated: 2026-08-17

## Commit Conventions
- Format: **Conventional Commits**. Evidence: `.husky/commit-msg` runs `npx commitlint --edit "$1"`, and recent history uses it (`style(frontend): apply prettier formatting to jest.setup.js`, `test(frontend): add comprehensive snapshot tests for all screens`, `fix(landing): prevent horizontal scroll / overflow on mobile`).
  - **Caveat:** no `commitlint` config file exists and neither `package.json` lists `@commitlint/*` or `lint-staged` as a dependency, so the hook likely no-ops locally. Treat Conventional Commits as the project's convention, not as a mechanically enforced gate — and note that some history entries are plain sentences (`Enhance README with Three.js and React Three Fiber badges`).
- Types in observed use: `feat`, `fix`, `test`, `style`, `chore`. Standard additions (`docs`, `refactor`, `perf`, `build`, `ci`) are consistent with the convention.
- Scopes: free-form, lowercase, naming the affected area. Observed: `frontend`, `landing`. Natural scopes for this repo: `backend`, `frontend`, `auth`, `passkey`, `employee`, `department`, `db`, `infra`, `ci`, `docs`.
- Subject: imperative mood, no trailing period, lowercase after the type/scope prefix.
- Body: explain *why* alongside *what* when the change is non-obvious; wrap comfortably (the repo's Prettier printWidth is 160 but commit bodies should stay ~72–100 chars).
- Footer / sign-off: TBD — no sign-off (`Signed-off-by`), issue-reference, or co-author convention is visible in history. PMO is GitHub Issues (`.factory/config.json`), so `Refs #<issue>` / `Closes #<issue>` is the natural fit if the team adopts one.
- Branch naming (from merged PR titles): `<type>/<short-slug>` — e.g. `feat/frontend-snapshot-tests`, `fix/fix-index-landing`, `chore/prettier-format`. Base branch is `master`.
- Pre-commit: `.husky/pre-commit` runs `npx lint-staged` (same caveat — no config committed). Run Prettier (`npm run format` / `cd frontend && npm run format`) before committing so CI's formatting job stays green.
- Never commit `backend/config.properties`, `.factory/.env`, `node_modules/`, `backend/target/`, or `frontend/build/`.
