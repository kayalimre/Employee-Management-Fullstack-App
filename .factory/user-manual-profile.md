# User Manual Profile

<!-- factory-version: 1.3.0 -->

> user-manual agent tuning. Read by the `user-manual` agent.
> Last updated: 2026-08-17

## Audience
- End users of the Employee Management web application: HR/administrative staff who manage employee and department records, and any employee who signs in to view or update their own profile. Non-technical — no JVM, database, or deployment vocabulary.

## Language(s)
- English (the entire UI is authored in English; no i18n framework is present).

## Platforms
- Web browser only (React SPA served at `http://localhost:3000` locally, or behind nginx in a deployed environment). Modern Chrome/Firefox/Safari/Edge per the CRA `browserslist`. Passkey sign-in requires a browser and device with WebAuthn support. No mobile or desktop app exists.

## Tone
- Plain, friendly, and instructional. Second person ("you"), short numbered steps, one action per step. Name the on-screen control exactly as the UI labels it.

## Coverage (from the screens present in `frontend/src/components/`)
- Landing page, sign in (`Login`), passkey sign-in, register, register a passkey, manage passkeys (`Passkeys`), forgot/reset password (`VerifyUsername` → `ResetPassword`), dashboard with charts (`Dashboard`, `QuickActions`), employee list/add/edit (`EmployeeList`, `EmployeeForm`), department list/add/edit (`DepartmentList`, `DepartmentForm`, `NewDepartmentForm`), profile (`Profile`), and what the "page not found" screen means.
- Explain passkeys in user terms (a fingerprint/face/device unlock replacing a password), including what happens on a device that does not support them.
- Include a short troubleshooting section for the visible failure states the app surfaces via toast notifications, plus the first-load delay the app itself warms up against.

## Exclusions
- No API endpoints, env vars, source paths, database schema, or deployment instructions.
- Never document credentials or seed accounts.
- Screenshot policy: TBD — the agent does not capture screenshots; note where one would help if the team wants to add them.
