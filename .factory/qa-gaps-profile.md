# QA Gaps Profile

<!-- factory-version: 1.3.0 -->

> qa-gaps agent tuning. Read by the `qa-gaps` agent.
> Last updated: 2026-08-17

## Gap categories
- **Missing flows** — a UI affordance with no backend endpoint (or the reverse); a screen reachable with no way back; an action with no success/failure feedback.
- **Unhandled errors** — axios calls in `frontend/src/services/*.js` without error handling, promises without `.catch`, backend paths that escape the `exception/` handlers into a raw 500, missing Bean Validation on request DTOs.
- **Dead ends** — routes not registered in `src/App.js`, components never rendered, endpoints with no frontend caller, `packages/` code that is compiled by nothing (report as dead code, not as a live gap).
- **Spec mismatches** — controller/DTO surface diverging from the root `openapi.yaml`; frontend service calls hitting paths or shapes the spec does not describe.
- **Auth gaps** — endpoints without an explicit authorization rule, screens not wrapped in `ProtectedRoute`, IDOR on employee/department ids, WebAuthn ceremony steps missing state/timeout validation.
- **State/consistency gaps** — deleting a department that still has employees, stale dashboard data after a mutation, session/JWT expiry not surfaced to the user.
- **Test coverage gaps** — flows with no backend JUnit test and no frontend Jest test.

## API spec location
- Root `openapi.yaml` (hand-maintained, authoritative for review purposes).
- springdoc-openapi-ui 1.7.0 generates the live spec and Swagger UI from the running backend — compare the two; a drift between hand-written `openapi.yaml` and the generated spec is itself a finding.
- Controllers to walk: `AuthController`, `EmployeeController`, `DepartmentController`, `PasskeyController`, `HealthController`, `HomeController` (`backend/src/main/java/com/example/employeemanagement/controller/`).
- Frontend callers to cross-check: `frontend/src/services/{authService,employeeService,departmentService,passkeyService}.js`.

## Severity scheme
- **Critical** — authentication/authorization bypass, data exposure, data loss.
- **High** — a core flow (employee/department CRUD, login, passkey login) is broken or unreachable; spec mismatch that would break a consumer.
- **Medium** — unhandled error paths, missing validation, missing user feedback, coverage gaps on important flows.
- **Low** — cosmetic dead ends, documentation/spec drift with no functional impact, stale duplicated code.

## Error-code matrix defaults
- Per route, check the "expected-error-code handled?" question against `claude-plugin/references/error-code-matrix.md`. In-scope by default for this project:
  - `400` — Bean Validation failures on request DTOs (`dto/*RequestDto`).
  - `401` — missing, malformed, or expired JWT; failed WebAuthn assertion.
  - `403` — authenticated but not permitted for the resource.
  - `404` — unknown employee/department/user id.
  - `409` — duplicate username/email, duplicate department name, passkey already registered.
  - `422` — only where the API actually uses it; otherwise record N/A.
  - `500` — unhandled exceptions escaping `exception/`.
- **Out of scope by default:** `429` (no rate limiting is implemented anywhere in the backend — flag the *absence* of rate limiting on auth endpoints as a security-adjacent gap instead). `502/503`/timeout apply to the compose/Kubernetes/nginx deployment path, not to a single local process.
- For each in-scope code per route, verify both that the backend can return it and that the frontend handles it (`src/utils/apiError.js` + the calling component's toast/UI state).
