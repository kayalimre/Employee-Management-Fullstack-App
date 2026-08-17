# Developer Profile

<!-- factory-version: 1.3.0 -->

> developer-agent tuning (stack/structure come from project-profile).
> Last updated: 2026-08-17

## Coding Style

**Java (backend)**
- Java 11 language level (pom `source`/`target` 11) — no records, no `var` in fields, no pattern matching for `instanceof`. Compiled with `-parameters`.
- Standard Spring Boot layering: `controller` -> `service` -> `repository`; `model` = JPA entities, `dto` = request/response payloads. Controllers accept/return DTOs, never entities directly (existing `*RequestDto` / `*ResponseDto` pairs set the pattern).
- Lombok is available (`provided` scope) — follow whatever the neighbouring class already does rather than introducing a new style.
- Constructor injection with `@Autowired`-free constructors where the surrounding code already does so; validation via `spring-boot-starter-validation` annotations on DTOs.
- Exceptions go through `exception/` (global handler + custom types), not ad-hoc `try/catch` in controllers.
- Package root is `com.example.employeemanagement`.

**JavaScript / React (frontend)**
- Functional components with hooks only. Files are `.js` (components/services/hooks) — snapshot *tests* use `.jsx`.
- Prettier is the formatter of record (`.prettierrc`): printWidth 160, 2-space indent, single quotes, semicolons, `trailingComma: "es5"`, `arrowParens: "avoid"`. Run `npm run format` rather than hand-formatting.
- API access goes through `frontend/src/services/*.js` (axios) — components must not call axios directly. Errors are normalized through `src/utils/apiError.js` and surfaced with `src/utils/toast.js` (react-toastify).
- Auth state via `src/hooks/useAuth.js`; route guarding via `components/ProtectedRoute.js`.
- Styling is mixed MUI (`sx` / theme in `src/theme.js`) + Tailwind utility classes — match the file you are editing; do not convert one to the other opportunistically.
- WebAuthn/passkey browser plumbing lives in `src/utils/webauthn.js` and `src/services/passkeyService.js`.

**General**
- Externalize user-facing strings where the surrounding code already does; prefer existing design tokens (`src/theme.js`, Tailwind config) over new hard-coded values.
- Document non-obvious public APIs (Javadoc on backend service/controller methods, JSDoc on exported frontend helpers).
- Never hard-code credentials, hosts, or secrets: backend reads env vars via `application.properties`; the untracked `backend/config.properties` (template `backend/example_config.properties`) is the local override mechanism.

## Conventions
- **Canonical backend source is `backend/src/main/java/com/example/employeemanagement/`. Never edit `packages/`** — `packages/java/employeemanagement/` is a stale duplicated snapshot with no build file, compiled by nothing. Editing it silently does nothing and creates drift. If a task explicitly asks to sync `packages/`, do it as a separate, clearly stated step.
- New backend endpoint: add/extend a controller in `controller/`, business logic in `service/`, persistence in `repository/`, payloads in `dto/`; then update the root `openapi.yaml` so the hand-maintained contract stays true (springdoc also exposes a live Swagger UI).
- New frontend screen: component in `frontend/src/components/`, API calls in a matching `frontend/src/services/*Service.js`, route wired in `src/App.js`, guarded with `ProtectedRoute` when it requires auth.
- Schema changes: Hibernate runs `ddl-auto=update`, but the SQL scripts in `databases/sql/` (and `data.sql`) are the documented schema/seed — update them alongside entity changes.
- Tests live in `backend/src/test/java/com/example/employeemanagement/` and `frontend/__tests__/` (behavior) / `frontend/__tests__/snapshots/` (snapshots). Authoring tests is the unit-test-engineer's job; leave existing snapshots alone unless the task is about them.
- Before reporting a task complete, build clean per `.factory/build-profile.md` (backend `mvn test` and/or frontend `npm test` for the side you touched) — and remember root `npm test` starts a dev server, it does not run tests.
- Commit messages follow Conventional Commits (see `.factory/commit-profile.md`).
