# CLAUDE.md

This file provides guidance for the AI assistant working in this repository.

## Project Overview

Employee Management Fullstack App — an employee and department management system built as a Spring Boot REST API plus a React single-page frontend. Users sign in with a password (JWT) or a WebAuthn passkey, then manage employee records, departments, and their own profile, with a chart-based dashboard over the data.

The repository holds two independently built applications, `backend/` and `frontend/`, alongside the infrastructure that ships them: Docker Compose, Kubernetes manifests (default, blue/green, canary, production), Terraform, nginx, and both a GitHub Actions workflow and a Jenkins pipeline.

## Build & Run

The backend builds with Maven (`cd backend && mvn clean install`, tests via `mvn test`, run via `mvn spring-boot:run` on port 8080). The frontend builds with npm/Create React App (`cd frontend && npm ci && npm run build`, tests via `npm test`, dev server via `npm start` on port 3000). Root `npm run dev` runs both at once, and `docker compose -f docker-compose.yml up --build` brings up the pair with MySQL and MongoDB. A `Makefile` wraps the common targets.

Three things to know before running anything: root `npm test` is wired to start the frontend dev server rather than run tests; root `npm install` is overridden by a custom script that also builds the backend; and the committed `backend/mvnw` wrapper is broken, so use the system `mvn` (which is what CI and the Makefile use anyway). The exact commands, plus the rest of the known traps, live in `.factory/build-profile.md`.

The backend reads its configuration from environment variables (`MYSQL_*`, `JWT_SECRET`, and the optional `MONGO_URI` / `WEBAUTHN_*` settings). Copy `backend/example_config.properties` to `backend/config.properties` for local overrides — that file is untracked and must stay that way.

## Module Architecture

`backend/` is a conventional layered Spring Boot service under `com.example.employeemanagement`: `controller/` (auth, employee, department, passkey, health, home) delegates to `service/`, which uses `repository/` over JPA entities in `model/`. Request and response payloads are DTOs in `dto/`, so entities never cross the API boundary. Authentication lives in `security/` (JWT) and `webauthn/` (FIDO2 passkey ceremonies), with cross-cutting setup in `config/` and error mapping in `exception/`.

`frontend/` is a Create React App SPA. Screens are in `src/components/` (with a Three.js hero in `components/hero3d/`), all HTTP access is funnelled through `src/services/*.js`, auth state through `src/hooks/useAuth.js`, and shared helpers — error normalization, toasts, WebAuthn plumbing — through `src/utils/`.

Supporting directories: `databases/` holds the SQL and MongoDB schema and seed scripts, `scripts/` holds build and deployment shell helpers, and the root `openapi.yaml` is the hand-maintained API contract that must be updated whenever the REST surface changes. The team's own deeper write-up is in `ARCHITECTURE.md`.

**One trap worth knowing:** `packages/java/` is a stale duplicated snapshot of the backend source. It has no build file, nothing compiles it, and edits there have no effect. The canonical backend source is `backend/src/main/java/`.

## Technology Stack

Backend: Java (compiled to 11, CI runs JDK 17), Spring Boot 2.7.5 with Spring Data JPA, Spring Security, and Bean Validation; MySQL 8 as the primary store with MongoDB configured but currently unused; jjwt for tokens, Yubico's `webauthn-server-core` for passkeys, Lombok, and springdoc for Swagger UI. Tests are JUnit 5 against an in-memory H2 database.

Frontend: React 18 with React Router 6, MUI 6 and Tailwind for styling, axios for HTTP, Chart.js for the dashboard, and Three.js via `@react-three/fiber` for the landing page. Tests are Jest with Testing Library, including a snapshot suite.

Tooling: Prettier formats the codebase, Husky hooks cover pre-commit and commit-message checks, and commits follow Conventional Commits. The default branch is `master`.

## Configuration

Machine-readable config for the adFactory dev-lane agents lives in the **`.factory/` profiles**
(`project-profile.md` plus the per-agent profiles), not here. Run `/factory:initialize` to regenerate them.

**External tooling.** Jira, Confluence, Azure DevOps, GitHub, GitLab, and Bitbucket are reached through adFactory's **bundled MCP servers**, resolved from `.factory/config.json` (roles `pmo` / `git` / `docs` → providers; credentials in `.factory/.env`, never committed). The **git** role accepts `azure-devops` | `github` | `gitlab` | `bitbucket`; **PMO** accepts `jira` | `azure-devops` | `github` | `gitlab`; **docs** accepts `confluence` | `azure-devops` (Wiki). One resource can back several roles (e.g. `github` for PRs + Issues; `azure-devops` for PRs + work items + Wiki). Use the MCP for the role a task needs — *which/when* is configured here; the *how* (tool names) comes from the servers themselves.

This project is configured with GitHub cloud (`kayalimre/Employee-Management-Fullstack-App`) backing both the git and PMO roles; there is no docs provider.

---

*Prose generated by `/factory:initialize`. Edit freely — agents read their config from `.factory/`, so changes here are safe.*
