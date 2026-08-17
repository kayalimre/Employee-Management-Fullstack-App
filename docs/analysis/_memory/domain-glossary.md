---
name: domain-glossary
description: Business domain, key concepts, term glossary
updated: 2026-08-17
rev: 0
author: aliemre
last-sync: never
synced-rev: 0
---

# Domain & Glossary

## Domain
Employee and department management for a small/medium organization. The application provides
CRUD workflows over employees and departments, a metrics dashboard (headcount, age distribution,
growth trends), and account features: JWT-based authentication plus WebAuthn passkey sign-in
and passkey management. Seeded demo data ships with the app. This is a personal/learning
project demonstrating an enterprise-style full stack (React + Spring Boot + MySQL/MongoDB +
Docker/Kubernetes/Terraform/CI-CD).

## Glossary
| Term | Definition |
|------|------------|
| Employee | Core entity: a person record with personal details and a department assignment |
| Department | Organizational unit grouping employees; target of employee assignment |
| Dashboard | Metrics view rendering employee counts, age distribution, and growth charts (Chart.js) |
| Passkey | WebAuthn credential enabling passwordless sign-in; users can add/rename/delete passkeys |
| JWT | JSON Web Token issued at login; attached by the frontend service layer to authenticated API calls |
| Seed data | Demo employees/departments loaded from `data.sql` / `databases/` at startup |
