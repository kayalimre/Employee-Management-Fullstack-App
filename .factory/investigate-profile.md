# Investigate Profile

<!-- factory-version: 1.3.0 -->

> investigate tuning. Read by the `investigate` agent.
> Last updated: 2026-08-17

## RCA Conventions
- Classification taxonomy: product defect | test defect | environment/setup.
- Where to look first, by failure shape:
  - **API case failing (4xx/5xx unexpected)** → `backend/src/main/java/com/example/employeemanagement/controller/` then `service/` then `repository/`; check `exception/` for how the status was mapped, and the DTO validation annotations for a `400`.
  - **401/403 anywhere** → `security/` (JWT filter/provider), `config/CorsConfig.java`, and whether the test obtained/attached a token in setup. Expired/short-lived tokens are a common test defect.
  - **Passkey/WebAuthn failure** → `webauthn/` + `PasskeyController`, and the `webauthn.rp-id` / `webauthn.allowed-origins` / `webauthn.allow-origin-port` properties. An origin/rp-id mismatch against the URL under test is environment, not product.
  - **Frontend behavior failure** → `frontend/src/services/*.js` (axios call shape), `src/utils/apiError.js` (error normalization), then the component. Check the request actually reached `:8080` — CORS and base-URL misconfiguration masquerade as UI bugs.
  - **Jest snapshot mismatch** → compare against `frontend/__tests__/snapshots/__snapshots__/*.snap`; an intentional UI change with a stale snapshot is a *test defect*, an unintended DOM change is a product defect.
  - **Backend test failing on datasource/context startup** → H2 + Spring test config; usually environment.
  - **Startup/timeout flakiness** → Spring Boot cold start and CRA first compile are slow; `frontend/src/utils/warmup.js` exists to mitigate hosted cold starts. Prefer an environment classification only when evidence shows the element/response did arrive late.
- **Always verify which source tree was changed.** If a "fix" was applied under `packages/java/`, it is compiled by nothing — the failure persists and the root cause is "edit landed in the stale duplicate, not `backend/src/main/java`."
- Version/toolchain trap: the pom targets Java 11 while CI uses temurin 17. Bytecode/toolchain errors that reproduce only locally are environment.
- Evidence expectations: cite `file:line` for product defects, the exact assertion/expected-vs-actual for test defects, and the specific missing/mismatched config value for environment findings. Never propose a fix without naming the file that must change.
- Depth: one root cause per failing case; group cases that share a single cause rather than repeating the analysis.

## RCA JSON Schema (RCA-{slug}.json)

```json
{
  "qa_report": "QAREPORT-{slug}",
  "generated_by": "investigate",
  "findings": [
    { "case": "TC-1", "root_cause": "...", "classification": "product | test | environment",
      "evidence": "file:line / condition", "suggested_fix": "..." }
  ]
}
```

## Sequential thinking
The bundled `sequentialthinking` reasoning tool (`factory-sequentialthinking` MCP) is **off by default**. Set `Status` to `enabled` to let the investigate agent use it — and even then only when the root cause is genuinely knotty enough to benefit (a clear-cut failure never triggers it). Any value other than `enabled`, or a missing section, reads as disabled.

Status: disabled
