# Reviewer Profile

<!-- factory-version: 1.3.0 -->

> reviewer-agent tuning.
> Last updated: 2026-08-17

## Project Review Context
Employee/department management system: a Spring Boot 2.7 REST API (MySQL via JPA, JWT + WebAuthn passkey auth, Spring Security, springdoc/OpenAPI) consumed by a React 18 CRA single-page app (MUI + Tailwind, axios service layer, Chart.js dashboards, a Three.js landing page). Two independently built apps in one repo, deployed as Docker images to Kubernetes (blue/green + canary manifests).

Review priorities for this project, in order:
1. **Auth correctness** — JWT issuance/validation and the WebAuthn ceremony flow are the highest-risk surfaces; a subtle mistake here is a full authentication bypass.
2. **API contract fidelity** — changes to controllers/DTOs must stay consistent with the hand-maintained root `openapi.yaml` and with the frontend `src/services/*.js` callers; drift between them is a recurring failure mode.
3. **Layering** — controller/service/repository separation, DTOs at the boundary (entities must not leak into responses).
4. **Duplicate-source trap** — any change made under `packages/java/` instead of `backend/src/main/java/` is dead code and must be flagged; `packages/` is a stale snapshot with no build file.

## Focus Areas

**Spring Boot / Java**
- Constructor injection over field injection; no business logic in controllers.
- Correct transaction boundaries (`@Transactional` on service methods that mutate), no lazy-loading outside the session.
- Bean Validation on request DTOs; consistent error handling through the `exception/` package rather than ad-hoc catch blocks.
- N+1 query risks from JPA relationships; `ddl-auto=update` means entity edits silently mutate the schema — check the `databases/sql/` scripts were updated too.
- Java 11 level only (pom targets 11 even though CI runs JDK 17) — flag records, sealed types, and other 12+ syntax.
- Null-safety and `Optional` handling on repository lookups.

**React / JavaScript**
- Hook rules and dependency arrays (`useEffect`/`useCallback`/`useMemo`); cleanup of subscriptions, timers, and in-flight axios requests on unmount.
- State that belongs in a parent should be hoisted, not duplicated; avoid derived state stored in `useState`.
- API calls only through `src/services/*.js`; errors normalized via `src/utils/apiError.js` and surfaced via `src/utils/toast.js` — not raw `alert`/`console.error`.
- Route protection: new authenticated screens must go through `ProtectedRoute`.
- Token/credential handling in the browser (where the JWT is stored, whether it leaks into logs or URLs).
- Three.js / `@react-three/fiber` components: dispose of geometries/materials, avoid re-creating objects each render, guard for WebGL-unavailable environments.
- Chart.js instances cleaned up on re-render; large lists rendered efficiently.
- Prettier formatting compliance (`.prettierrc`, printWidth 160) — formatting-only noise in a diff should be called out separately from logic changes.

**Cross-cutting**
- `openapi.yaml` updated when the REST surface changes.
- No secrets, hosts, or credentials hard-coded; backend config must come from env vars.
- CORS changes (`config/CorsConfig.java`) and WebAuthn origin config reviewed as security-relevant, not cosmetic.
- Kubernetes/Terraform/compose edits reviewed for consistency across the default/blue/green/canary/production manifest sets.

## Flags
- Performance-critical: false — this is a CRUD business app; correctness and security outweigh micro-optimization. (Exceptions: the Three.js landing page and dashboard chart rendering, where render-loop cost is worth a look.)
- Accessibility-required: TBD — the frontend tests already query by role/label (`getByRole`, `getByLabelText`), which implies accessible markup is valued, but no formal a11y standard is declared for this project. Set to `true` and name the target (e.g. WCAG 2.1 AA) if the team commits to one.
