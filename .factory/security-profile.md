# Security Profile

<!-- factory-version: 1.3.0 -->

> security-agent tuning.
> Last updated: 2026-08-17

## Security Concerns

**Authentication & tokens**
- `io.jsonwebtoken:jjwt:0.9.1` is a very old, effectively unmaintained JWT library (the modern line is `jjwt-api`/`jjwt-impl`/`jjwt-jackson` 0.11+). Check signing algorithm, key length, expiry validation, and that `alg: none` / algorithm confusion is impossible.
- `jwt.secret` comes from the `JWT_SECRET` env var (`backend/src/main/resources/application.properties`) — verify no default/fallback value is ever compiled in, and that the secret is not logged.
- Token storage on the frontend (`src/hooks/useAuth.js`, `src/services/authService.js`): check for XSS-exposed storage, tokens in URLs, and tokens leaking into logs or error toasts.

**WebAuthn / passkeys** (`backend/.../webauthn/`, `PasskeyController`, `frontend/src/utils/webauthn.js`)
- `webauthn.rp-id`, `webauthn.allowed-origins`, and especially `webauthn.allow-origin-port` (default `false` — flag any change to `true`) must not be loosened.
- Challenge generation/storage, ceremony timeout (`webauthn.ceremony-timeout-seconds`, default 300), replay protection, signature-counter handling, and credential-to-user binding.
- `databases/sql/08_webauthn_credentials.sql` — credential storage schema.

**Spring Security / API surface**
- `config/CorsConfig.java`: wildcard origins combined with credentials is a critical finding.
- Endpoint authorization: every controller (`Auth`, `Employee`, `Department`, `Passkey`, `Health`, `Home`) must have an intentional permit/authenticated rule; watch for broadly permitted paths.
- BCrypt password encoding (`spring-security-crypto`), password reset flow (`ResetPassword`, `VerifyUsername` screens) — check for user enumeration and unauthenticated reset.
- IDOR on employee/department resources (path-id access without ownership/role check).
- `springdoc-openapi-ui` Swagger UI exposure in production.

**Secrets & configuration**
- `backend/config.properties` is untracked (only `example_config.properties` is committed) — verify it stays out of git and that `.gitignore` continues to cover it.
- `docker-compose.yml` contains hardcoded dev credentials (`MYSQL_ROOT_PASSWORD: password`, root/password in the datasource URL) — acceptable for local dev, must never propagate to any deployed manifest.
- `kubernetes/secrets-template.yaml` — confirm it stays a template and no real values are committed; check `configmap*.yaml` for credentials.
- Seed data: `data.sql`, `databases/sql/04_seed_data.sql`, `databases/mongo/03_seed_data.js` — flag any default/admin credentials that could survive into a real environment (`config/DataInitializer.java` too).
- CI: `.github/workflows/ci.yml` uses `secrets.GITHUB_TOKEN` for GHCR; `Jenkinsfile` (21KB) should be scanned for inline credentials.
- Terraform (`terraform/`), `aws/`, `nginx/` configs — check for embedded keys, permissive security groups, and public buckets.

**Data layer**
- `spring.jpa.hibernate.ddl-auto=update` in production is a schema-integrity risk worth flagging.
- `spring.jpa.show-sql=true` can leak data into logs.
- Raw SQL in `databases/sql/*.sql` and any native queries — injection review.
- MongoDB is configured but reportedly unused; an exposed/unauthenticated Mongo URI is still a real exposure.

**Dependencies**
- Backend: Spring Boot 2.7.5 (out of OSS support), jjwt 0.9.1, mysql-connector-j 8.0.33, Yubico webauthn-server-core 2.6.0, Lombok 1.18.30, javafaker 1.0.2. The pom pins `jackson-bom.version=2.19.4` and overrides SNAPSHOT repositories to force released Jackson versions — verify those pins do not fall behind security releases.
- Frontend: react-scripts 5.0.1 (transitively pulls dated tooling), axios, three 0.169, MUI 6, jest 27.
- Container base images in `backend/Dockerfile`, `frontend/Dockerfile`, and `docker-compose.yml` (`mysql:8.0`, `mongo:6.0`).

## Scan Focus
- OWASP Top 10 emphasis: A01 Broken Access Control (IDOR, endpoint authorization), A02 Cryptographic Failures (JWT signing, password hashing, TLS/`MYSQL_SSL_MODE`), A05 Security Misconfiguration (CORS, `ddl-auto`, Swagger exposure, compose/k8s defaults), A07 Identification & Authentication Failures (JWT + WebAuthn ceremonies, password reset), A06 Vulnerable and Outdated Components (Spring Boot 2.7.5, jjwt 0.9.1).
- Secret patterns: JWT signing keys, `MYSQL_PASSWORD`, GitHub/GHCR tokens, AWS keys, private keys and `.pem` material, connection strings in `*.properties`, `*.yaml`, `*.tf`, `Jenkinsfile`, and shell scripts under `scripts/`.
- Dependency advisories: `backend/pom.xml` (and its lock-free transitive tree), `frontend/package-lock.json`, root `package-lock.json`.

## Exclusions
- `packages/` — a stale, non-buildable duplicate of the backend source; findings there are not shipped code. Report drift as a hygiene note, not as a live vulnerability.
- `node_modules/`, `backend/target/`, `frontend/build/`, `.idea/`, `img/` — generated, vendored, or non-code assets.
- `docker-compose.yml` and `databases/**` dev credentials are known local-development defaults: report once as "must not reach a deployed environment", do not re-raise per occurrence.
- Additional project-specific exclusions: TBD.
