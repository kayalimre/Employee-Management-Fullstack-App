---
name: architecture
description: Architecture, tech stack, main modules/services, repo structure
updated: 2026-08-17
rev: 0
author: aliemre
last-sync: never
synced-rev: 0
---

# Architecture & Tech Stack

## Tech stack
- **Frontend:** React 18.3 SPA (Create React App 5), React Router 6, Material UI 6 + Tailwind CSS 3.4, axios, Chart.js 4 (react-chartjs-2), Three.js + @react-three/fiber (landing visuals). Tests: Jest 27 + React Testing Library (behavior + snapshot specs under `frontend/__tests__/`). Prettier enforced.
- **Backend:** Spring Boot 2.7.5 on Java 11 (Maven) — Spring Data JPA, Spring Security, Validation, optional Spring Data MongoDB; jjwt 0.9.1 (JWT auth), Yubico webauthn-server-core 2.6.0 (passkeys), springdoc-openapi-ui (Swagger UI), Lombok. Tests: JUnit 5 + H2 + javafaker.
- **Data:** MySQL 8 primary datastore (JPA); optional MongoDB 6. Seeds in `data.sql` and `databases/` (sql + mongo).
- **Infra/CI:** Docker Compose (backend, frontend, mysql, mongodb), Kubernetes manifests (default/blue/green/canary/production), Terraform + AWS assets, nginx config, Jenkinsfile and GitHub Actions.

## Main modules / services
- `frontend/src/components/` — Dashboard (Chart.js metrics), Employee/Department CRUD (EmployeeList/Form, DepartmentList/Form), auth pages (Login, Register, ResetPassword, VerifyUsername), passkey management (Passkeys, PasskeyPromptDialog), Navbar/Footer, LandingPage.
- `frontend/src/services/` — axios data layer (`employeeService.js`, `departmentService.js`, `passkeyService.js`); default base URL is the Render-hosted backend (`https://employee-management-app-gdm5.onrender.com`), `passkeyService` honors `REACT_APP_API_BASE_URL`; JWT attached to authenticated calls. `src/utils/webauthn.js` drives the WebAuthn ceremonies.
- `backend/src/main/java/com/example/employeemanagement/` — Spring Boot service (port 8080): REST controllers, services, JPA repositories, security/JWT + WebAuthn config, HealthController. API documented in root `openapi.yaml` and via Swagger UI.

## Repo structure (brownfield — derived from scan)
- `frontend/` — React SPA (port 3000); tests in `frontend/__tests__/`
- `backend/` — Spring Boot service (Maven; tests in `backend/src/test/`)
- `databases/` — MySQL + MongoDB seed/init assets
- `openapi.yaml` + `scripts/openapi-client.sh` — API contract and generated-client helper
- `docker-compose.yml`, `kubernetes/`, `terraform/`, `aws/`, `nginx/`, `Jenkinsfile`, `Makefile`, `scripts/`
- `ARCHITECTURE.md`, `README.md`, `DEPLOYMENT.md`, `WIKI.md` — in-repo documentation

### Known repo quirks (verified)
- `backend/mvnw` is broken (wrapper properties live at repo root) — use system `mvn`; project needs JDK 11/17, not newer.
- `packages/java/` is a stale, unreferenced duplicate of the backend sources — never edit it.
- Root `package.json`: `npm test` actually starts the frontend dev server; `npm install` also builds the backend.
- `Makefile` `docker-up`/`docker-down` reference `docker-compose.yaml` but the file is `docker-compose.yml`.
- Frontend service files hardcode the Render backend URL; local development requires overriding it manually.
