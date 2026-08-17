# Build Profile

<!-- factory-version: 1.3.0 -->

> Build/run config. Read by /factory:build (inline), the developer (builds clean before completing), the unit-tester (runs tests), and the unit-test-engineer (compiles new tests).
> Last updated: 2026-08-17

## Commands

All commands are run from the project root unless a `cd` is shown.

**Backend (Maven — use the system `mvn`, NOT the wrapper):**
- Build: `cd backend && mvn clean install`
- Build (skip tests): `cd backend && mvn clean install -DskipTests`
- Compile only: `cd backend && mvn compile`
- Test: `cd backend && mvn test`
- Single test class: `cd backend && mvn test -Dtest=EmployeeManagementApplicationTests`
- Run: `cd backend && mvn spring-boot:run` (serves on `http://localhost:8080`)
- Clean: `cd backend && mvn clean`

**Frontend (npm, CRA + Jest):**
- Install: `cd frontend && npm ci`
- Build: `cd frontend && npm run build` (production bundle into `frontend/build/`)
- Test: `cd frontend && npm test` (`jest --config jest.config.js --runInBand`)
- Test (watch): `cd frontend && npm run test:watch`
- Coverage: `cd frontend && npm run test:coverage`
- Run: `cd frontend && npm start` (dev server on `http://localhost:3000`)
- Clean: `cd frontend && rm -rf build node_modules`

**Formatting:**
- Root: `npm run format` (`prettier --write "**/*.js"`)
- Frontend: `cd frontend && npm run format`
- Config: `.prettierrc` (printWidth 160, tabWidth 2, singleQuote, semi, trailingComma es5, arrowParens avoid), ignore list in `.prettierignore`
- Lint: no ESLint run script is wired; CRA's `eslintConfig` (`react-app`, `react-app/jest`) only applies during `react-scripts` build/start.

**Makefile shortcuts (working):** `make backend-build`, `make backend-run`, `make backend-test`, `make frontend-install`, `make frontend-build`, `make frontend-run`, `make frontend-test`, `make k8s-apply`, `make k8s-delete`, `make clean`.

**Full stack (dev):** `npm run dev` at the root (`concurrently` runs `mvn spring-boot:run` + frontend `npm start`).

### Command traps — read before invoking anything at the root
- **Root `npm test` does NOT run tests.** Root `package.json` defines `"test": "cd frontend && npm start"`, which launches the dev server and blocks. Never use it as a test command; use `cd frontend && npm test`.
- **Root `npm install` is overridden** by a custom `"install"` script (`cd frontend && npm install && cd ../backend && mvn install -DskipTests`), so a plain `npm install` at the root triggers a full backend build. Install dependencies per app instead.
- **`make docker-up` / `make docker-down` are broken**: the Makefile sets `DOCKER_COMPOSE_FILE := docker-compose.yaml` but the file on disk is `docker-compose.yml`. Use `docker compose -f docker-compose.yml up --build` / `down` directly.
- Backend runtime config comes from env vars (`MYSQL_HOST`, `MYSQL_PORT`, `MYSQL_DB`, `MYSQL_USER`, `MYSQL_PASSWORD`, `MYSQL_SSL_MODE`, `JWT_SECRET`, optional `MONGO_URI`, `WEBAUTHN_*`), optionally imported from an untracked `backend/config.properties` (template: `backend/example_config.properties`). `mvn test` uses H2 and does not need MySQL; `spring-boot:run` does.
- **`backend/mvnw` is broken — do not use it.** The wrapper script resolves `./.mvn/wrapper/maven-wrapper.properties` relative to `backend/`, but that directory does not exist (the wrapper properties live at the *repo root* `.mvn/wrapper/`, and there is no `mvnw` script at the root). Running it fails with `cannot read distributionUrl property`. Use the system `mvn`, which is also what CI, the `Makefile`, and the root `package.json` scripts all use.
- JDK: the pom targets Java 11 (`java.version`, compiler source/target 11) on Spring Boot 2.7.5, while CI uses temurin 17. A much newer default JDK on a developer machine (e.g. JDK 21+/26) will not build this stack — check `mvn -v`'s reported Java version first when a build fails with toolchain, bytecode, or reflective-access errors, and switch `JAVA_HOME` to a JDK 11 or 17 install.

## Variants / Configurations
- **Compile-time variants:** none defined. There are no Maven profiles and no Spring profiles (`spring.profiles.*` is absent from `application.properties`). Frontend has only CRA's implicit development (`npm start`) vs production (`npm run build`) modes.
- **Container/deploy targets (distinct from build variants):**
  - local: `docker compose -f docker-compose.yml up --build` (backend 8080, frontend 3000->80, mysql 3306, mongodb 27017)
  - Kubernetes (`kubernetes/`): default, `-blue`, `-green`, `-canary`, and `-production` manifest sets, plus hpa/pdb/rbac/network-policy and `secrets-template.yaml`
  - deployment helper scripts in `scripts/` (`deploy-k8s.sh`, `deploy-blue-green.sh`, `deploy-canary.sh`, `promote-canary.sh`, `rollback-*.sh`, `switch-blue-green.sh`)
  - Terraform (`terraform/`) and AWS assets (`aws/`) for the cloud footprint

## Outputs
- Backend jar: `backend/target/employee-management-app-0.0.1-SNAPSHOT.jar` (pattern `backend/target/*.jar`)
- Backend compiled classes: `backend/target/classes/`
- Backend test reports: `backend/target/surefire-reports/*.xml` (archived by CI)
- Frontend production bundle: `frontend/build/`
- Frontend coverage (when `test:coverage` is run): `frontend/coverage/`
- Docker images: `employee-management-app-backend:latest`, `employee-management-app-frontend:latest` locally; `ghcr.io/<owner>/employee-management-backend|frontend:{sha,latest}` from CI
