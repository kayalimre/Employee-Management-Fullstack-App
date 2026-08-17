# Dev Manual Profile

<!-- factory-version: 1.3.0 -->

> dev-manual agent tuning. Read by the `dev-manual` agent.
> Last updated: 2026-08-17

## Audience
- Developers joining or contributing to this repository: Java/Spring Boot backend engineers and React frontend engineers, plus whoever touches the Docker/Kubernetes/Terraform deployment path. Assume general professional experience, no prior knowledge of this codebase.

## Depth
- Practical and task-oriented: enough architecture to orient, then concrete recipes (how to run it locally, how to add an endpoint, how to add a screen, how to run and extend tests, how to deploy). Cite real file paths from this repo rather than generic examples.

## Sections to include
- Getting started: prerequisites (JDK — pom targets 11, CI uses 17; Node 18 per CI; Docker; MySQL 8 / MongoDB 6 or compose), cloning, `backend/config.properties` from `backend/example_config.properties`, required env vars (`MYSQL_*`, `JWT_SECRET`, `WEBAUTHN_*`).
- Running locally: root `npm run dev`, per-app commands, `docker compose -f docker-compose.yml up --build`, working Makefile targets — and the known traps (root `npm test` starts the dev server; `make docker-up` references the wrong compose filename).
- Architecture overview: backend layering (controller → service → repository, DTO boundary, `exception/`, `security/`, `webauthn/`, `config/`), frontend structure (components, services, hooks, utils, theme), data stores (MySQL primary via JPA; MongoDB configured but unused). Reference the team's own root `ARCHITECTURE.md` rather than duplicating it wholesale.
- Auth deep-dive: JWT flow and the WebAuthn/passkey ceremonies (start/finish), since these are the least self-evident parts of the system.
- API contract: root `openapi.yaml`, springdoc Swagger UI, `scripts/openapi-client.sh`, and the rule that controller changes require an `openapi.yaml` update.
- Database: `databases/sql/` and `databases/mongo/` scripts, `data.sql`, `config/DataInitializer.java`, and the `ddl-auto=update` implication.
- Testing: JUnit 5 + H2 backend suite, Jest + Testing Library frontend suite incl. snapshots, how to run each, where reports land.
- Conventions: Prettier config, Conventional Commits, Husky hooks (and that their configs are absent), branch naming, CI expectations.
- Deployment: Docker images, `kubernetes/` (default/blue/green/canary/production), `scripts/deploy-*.sh`, Terraform, nginx, `Jenkinsfile` vs GitHub Actions.
- **Gotchas section (must include):** `packages/` is a stale duplicate of the backend source and is compiled by nothing — never edit it.

## Exclusions
- Do not document `node_modules/`, `backend/target/`, `frontend/build/`, `.idea/`, or `img/`.
- Do not treat `packages/` as an API surface — mention it only as the gotcha above.
- Do not restate the full README (47KB) or root `ARCHITECTURE.md`; link to them.
- No credentials or real secret values, ever — reference the env var names and the example config file.
