---
name: decisions-integrations
description: ADR-like decisions, external integrations/systems, constraints/NFR hints
updated: 2026-08-17
rev: 0
author: aliemre
last-sync: never
synced-rev: 0
---

# Decisions & Integrations

## Decisions (ADR-like)
| ID | Decision | Rationale | Date |
|----|----------|-----------|------|
| D-001 | Dual datastore: MySQL primary (JPA), MongoDB optional | Inherited from upstream demo design; showcases polyglot persistence | TBD (inherited) |
| D-002 | Passwordless option via WebAuthn passkeys alongside JWT login | Modern auth demonstration (Yubico webauthn-server) | TBD (inherited) |

## External integrations / systems
| System | Purpose | Notes |
|--------|---------|-------|
| Render | Hosts the demo backend | Default API base URL in frontend services |
| Vercel | Hosts the demo frontend | https://employee-manage-app.vercel.app |
| GitHub | Source hosting, PRs, issues (adFactory git + PMO roles) | kayalimre/Employee-Management-Fullstack-App |

## Constraints / NFR hints
- Backend requires Java 11 (Spring Boot 2.7.x); newer JDKs break the build.
- jjwt 0.9.1 and Spring Boot 2.7.5 are dated — security posture of auth stack should be revisited before any real-world use.
- Free-tier Render hosting cold-starts the backend (repo includes cold-start mitigation work).
