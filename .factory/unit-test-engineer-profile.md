# Unit Test Engineer Profile

<!-- factory-version: 1.3.0 -->

> unit-test-engineer tuning — the AUTHORING side (writes/updates tests).
> Last updated: 2026-08-17

## Test Frameworks
- Backend: JUnit 5 (Jupiter) via `spring-boot-starter-test` (JUnit 5 confirmed — no JUnit 4 usage in the tree), Spring Boot test support (`@SpringBootTest`, MockMvc), Mockito (bundled with the starter), H2 in-memory DB, javafaker for synthetic data.
- Frontend: Jest 27 (`jsdom`) + @testing-library/react 13 + jest-dom matchers + user-event, babel-jest transform, `identity-obj-proxy` for CSS modules, `axios` explicitly un-ignored in `transformIgnorePatterns`, global setup in `frontend/jest.setup.js`.

## Authoring Conventions
- **Locations:** backend tests in `backend/src/test/java/com/example/employeemanagement/`; frontend behavior tests in `frontend/__tests__/`; frontend snapshot tests in `frontend/__tests__/snapshots/`. Never write tests under `packages/` (stale, non-compiled duplicate).
- **Naming:**
  - Backend classes: `<Area>Tests.java` / `<Area>ApiIntegrationTest.java` (existing: `APITests`, `AdditionalAPITests`, `AuxAPITests`, `BackendAPITests`, `FullAPITests`, `UtilitiesTests`, `PasskeyApiIntegrationTest`, `*ApplicationTests`). `APITestSuite.java` aggregates suites.
  - Frontend behavior specs: `<Component>.test.js` (one `.spec.js` exists — `Dashboard.spec.js`; prefer `.test.js` for new files).
  - Frontend snapshot specs: `<Component>.snapshot.test.jsx` under `__tests__/snapshots/`.
- **Structure:** Arrange-Act-Assert. Backend: `@Test` methods with descriptive camelCase names; frontend: `describe('<Component>', ...)` + `it('...')` phrased as behavior.
- **Mocking strategy — boundaries only.**
  - Backend: mock repositories/external clients with Mockito when unit-testing services; prefer `@SpringBootTest` + MockMvc + H2 for controller/API-level tests (matching the existing style) rather than mocking Spring internals.
  - Frontend: mock the axios-based service modules in `src/services/*.js` (`jest.mock('../src/services/employeeService')`) — do not mock React internals or reach into component state. Stub `window.navigator.credentials` for WebAuthn/passkey paths.
- **Queries (frontend):** accessible queries only — `getByRole`, `getByLabelText`, `getByText`, and `findBy*` for async. This matches the existing suite. Avoid container/DOM traversal and class selectors.
- **Fixtures:** backend uses javafaker for generated data and H2 for persistence; there is no shared fixture/factory module on either side yet. Keep test data local to the test unless a shared builder becomes clearly justified. Synthetic-data values must come from the reserved, non-routable ranges in `claude-plugin/references/safe-synthetic-data.md` (`@example.com`, `555-01NN`, `192.0.2.0/24`, `4242…`) — never a real address, phone, or host.
- **Snapshots:** update a `.snap` only when the corresponding UI change is intentional and part of the current task; state explicitly in the report which snapshots were re-recorded and why. Never blanket-run `jest -u`.
- **Determinism:** no real network calls, no wall-clock or `Math.random()` dependence (freeze/inject instead), no reliance on test execution order. Frontend runs serial (`--runInBand`) — do not write tests that depend on that.
- **Formatting:** Prettier config applies to test files too (printWidth 160, single quotes, semi, `arrowParens: "avoid"`).
- Confirm authored tests compile/run (`cd backend && mvn test -Dtest=<Class>`, `cd frontend && npx jest --config jest.config.js --runInBand <file>`) before handing off; judging the overall suite is the unit-tester's job.

## Coverage Priorities
- Authentication: JWT login/registration (`AuthController`, `authService.js`, `Login`/`Register` screens) and the WebAuthn/passkey ceremonies (`PasskeyController`, `webauthn/`, `passkeyService.js`, `LoginPasskey`/`RegisterPasskey`/`Passkeys`) — both success and rejection paths.
- Authorization boundaries on employee/department endpoints (unauthenticated and wrong-user access must fail).
- Employee and department CRUD, including validation failures and not-found handling (`exception/` mappings).
- Frontend service-layer error normalization (`utils/apiError.js`) and protected-route redirects (`ProtectedRoute`).
- Password reset / username verification flows.
- Remaining project-specific priorities: TBD.
