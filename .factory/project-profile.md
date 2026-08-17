# Project Profile

<!-- factory-version: 1.3.0 -->

> Shared project facts. Read first by every dev-lane agent.
> Last updated: 2026-08-17

## Overview
- Name: Employee-Management-Fullstack-App (root `package.json` name: `employee-management-app`)
- Type: fullstack application repository — one Spring Boot REST service + one React SPA in a single git repo (not a formal monorepo; each app has its own build)
- Platforms: backend (JVM/Spring Boot REST API), web (React single-page app)
- Description: Employee and department management system. Spring Boot backend exposing a REST API over MySQL (JPA) with JWT authentication and WebAuthn/passkey login; React 18 frontend (MUI + Tailwind + Chart.js + Three.js landing) consuming that API. Ships with Docker Compose, Kubernetes manifests (default/blue/green/canary/production), Terraform, and an nginx frontend image.

## Tech Stack
- Languages (tracked files, `git ls-files`): java 109, js 55, terraform 25, sql 17, yaml/yml 22, xml 20, jsx 17, shell 14, css 3
  - **Note:** the java count double-counts — roughly half of those files are a duplicated snapshot under `packages/java/` (see Structure). Canonical backend java lives only under `backend/src/`.
- Backend: Java (pom pins `java.version=11`, `maven-compiler-plugin` source/target 11) — **CI builds on JDK 17** (`.github/workflows/ci.yml` `setup-java` temurin 17). Use a JDK that satisfies both (17 toolchain compiling to 11 bytecode) unless the pom is changed.
- Backend frameworks: Spring Boot 2.7.5 (starter-web, data-jpa, data-mongodb, security, validation, devtools), Spring Data JPA + Hibernate (`ddl-auto=update`), Spring Security + BCrypt, jjwt 0.9.1 (JWT), Yubico `webauthn-server-core` 2.6.0 (passkeys/FIDO2), springdoc-openapi-ui 1.7.0 (Swagger UI), Lombok 1.18.30, javafaker 1.0.2
- Frontend frameworks: React 18.3, react-scripts 5.0.1 (Create React App), React Router 6.26, MUI 6 (+ Emotion), TailwindCSS 3.4 (+ postcss, autoprefixer), axios, Chart.js 4 / react-chartjs-2, Three.js 0.169 with @react-three/fiber + drei + postprocessing, react-toastify
- Data stores: MySQL 8 (primary, via JPA), MongoDB 6 (configured via `spring.data.mongodb.uri`, but the application.properties comment states no repository currently uses it)
- Build systems / package managers: Maven (backend — use the system `mvn`; the committed `backend/mvnw` wrapper is broken, see build-profile) + npm (frontend and root). `Makefile` wraps the common targets.
- Test frameworks: JUnit 5 / Jupiter via `spring-boot-starter-test` with H2 in-memory DB (backend); Jest 27 + @testing-library/react 13 + @testing-library/jest-dom + babel-jest, `jsdom` environment, snapshot tests (frontend)
- API contract: root `openapi.yaml` (hand-maintained) plus springdoc-generated Swagger UI served by the running backend
- CI: two systems — `.github/workflows/ci.yml` (format → backend tests → frontend tests → build/push GHCR images → deploy) and a large root `Jenkinsfile`
- Tooling: Prettier 3.3 (`.prettierrc`, printWidth 160, singleQuote, semi, trailingComma es5, arrowParens avoid); Husky hooks (`pre-commit` → `npx lint-staged`, `commit-msg` → `npx commitlint --edit`)
- Infrastructure: `docker-compose.yml` (backend + frontend + mysql + mongodb), `kubernetes/` (default, blue, green, canary, production variants), `terraform/`, `aws/`, `nginx/`, `.devcontainer/`

## Structure
- Apps:
  - `backend` -> `backend/` (Maven `com.example:employee-management-app`, Java package `com.example.employeemanagement`, Spring Boot service on port 8080)
  - `frontend` -> `frontend/` (npm `employee-management-system-frontend`, CRA dev server on port 3000, served by nginx in Docker on port 80)
- Backend internal layout (`backend/src/main/java/com/example/employeemanagement/`): `controller/` (Auth, Department, Employee, Health, Home, Passkey), `service/`, `repository/`, `model/`, `dto/`, `security/`, `webauthn/`, `config/` (CorsConfig, DataInitializer), `exception/`
- Frontend internal layout (`frontend/src/`): `components/` (20 screens/components + `hero3d/`), `services/` (auth, department, employee, passkey), `hooks/` (`useAuth`), `utils/` (apiError, toast, warmup, webauthn), `theme.js`
- Tests: `backend/src/test/java/com/example/employeemanagement/` (11 JUnit 5 classes); `frontend/__tests__/` (8 behavior specs) and `frontend/__tests__/snapshots/` (17 snapshot specs + `__snapshots__/*.snap`)
- Supporting directories: `databases/` (`sql/` schema+seed scripts, `mongo/` init scripts), `scripts/` (build/deploy/test shell helpers, `openapi-client.sh`), `kubernetes/`, `terraform/`, `aws/`, `nginx/`, `img/`, root `index.html` + `data.sql`
- **`packages/` is NOT a module.** `packages/java/employeemanagement/` is a duplicated snapshot of `backend/src/main/java/com/example/employeemanagement/` (verified: identical except it lacks `HealthController.java`), and `packages/script.js` / `packages/styles.css` are stray frontend snippets. There is no `pom.xml` or `package.json` under `packages/`, and nothing in the Maven or npm build references it — it is not compiled, not tested, and not shipped. Never edit or read it as source of truth; treat `backend/src/main/java` as canonical.

## Versioning
- File: root `package.json` (project-level aggregate) — per-app version files listed below
- Format: json (root + frontend), xml/maven (backend)
- Strategy: per-app (three independent versions, not kept in sync)
- Keys:
  - root `package.json` -> `version` (currently `1.0.1`)
  - `frontend/package.json` -> `version` (currently `1.1.0`)
  - `backend/pom.xml` -> `<version>` (currently `0.0.1-SNAPSHOT`)
- Container image tags are derived in CI from `${{ github.sha }}` + `latest`, not from these version fields.

## Learning History
- (empty — agents append dated entries here as they learn project-specific facts)
