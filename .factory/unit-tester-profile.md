# Unit Tester Profile

<!-- factory-version: 1.3.0 -->

> unit-tester tuning — the RUN side (the test command lives in build-profile).
> Last updated: 2026-08-17

## Test Frameworks
- Backend: JUnit 5 (Jupiter) via `spring-boot-starter-test`, with Spring Boot test slices/`@SpringBootTest`, MockMvc, H2 in-memory database (`test` scope), and javafaker for data generation. 11 test classes in `backend/src/test/java/com/example/employeemanagement/` (incl. `APITestSuite`, `PasskeyApiIntegrationTest`).
- Frontend: Jest 27 (`jsdom` environment) + @testing-library/react 13 + @testing-library/jest-dom + @testing-library/user-event, transformed by babel-jest. 8 behavior specs in `frontend/__tests__/` and 17 snapshot specs in `frontend/__tests__/snapshots/`.

## Coverage Targets
- TBD — no `coverageThreshold` is configured in `frontend/jest.config.js` and no JaCoCo/coverage plugin is present in `backend/pom.xml`. Frontend coverage can still be measured on demand via `npm run test:coverage`; backend coverage requires adding a plugin first.

## Run Conventions
- Backend (all): `cd backend && mvn test`
- Backend (single class): `cd backend && mvn test -Dtest=<ClassName>`; single method: `-Dtest=<ClassName>#<methodName>`
- Frontend (all): `cd frontend && npm test` — already `--runInBand` (serial); do not add `--watch` in an automated run.
- Frontend (single file): `cd frontend && npx jest --config jest.config.js --runInBand __tests__/Login.test.js`
- Frontend (by name): append `-t "<test name pattern>"`
- **Never invoke root `npm test`** — root `package.json` maps it to `cd frontend && npm start`, which launches the dev server and hangs.
- Backend tests use H2 and Spring's test context, so no MySQL/MongoDB instance is needed. If a test fails on datasource wiring, that is a configuration defect, not a missing service.
- Test reports: `backend/target/surefire-reports/*.xml` (also archived by GitHub Actions). Frontend coverage lands in `frontend/coverage/`.
- JDK note: the pom compiles to Java 11 (Spring Boot 2.7.5) while CI uses temurin 17 — a local failure that looks like a bytecode/toolchain error is environmental, report it as such (check `mvn -v`; a much newer default JDK will not build this stack). Also note `backend/mvnw` is broken and must not be used — always the system `mvn`.
- Parallelism: backend Maven surefire runs with defaults (no parallel config); frontend is pinned serial via `--runInBand`. Do not introduce parallel flags to chase speed.
- Snapshot failures: an assertion mismatch against `frontend/__tests__/snapshots/__snapshots__/*.snap` is a *result to report*, not something to "fix" by re-recording. Never run Jest with `-u` — deciding whether a snapshot is stale belongs to the unit-test-engineer.
- Flaky-test handling: TBD — no retry mechanism is configured. Re-run a suspected flake once, and report it as flaky with both outcomes rather than silently retrying to green.
