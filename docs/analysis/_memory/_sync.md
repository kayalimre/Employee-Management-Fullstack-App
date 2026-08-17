---
name: _sync
description: Docs/PMO sync state — content mappings only
updated: 2026-08-17
---

# Docs / PMO Sync

Connection details (base URL, space key, parent page, Jira project key) live in
`.factory/config.json`, resolved by ROLE — `docs.provider` then that provider's `resources.*` block
(same for `pmo.provider`);
credentials live in `.factory/.env`. Configure both with `/factory:initialize`.
This file records only the **content mapping** between the BA memory/artifacts and
the docs/PMO platform — page ids or wiki paths, issue keys, and last-sync state.

| Field | Value |
|-------|-------|
| Connector | none |
| MCP | factory-github (bundled with adFactory) — PMO role; no docs provider configured |
| Config overrides | none |

> Note: `docs.provider` is not set (docs stay local in the repo). `pmo.provider` is `github`
> (Issues on kayalimre/Employee-Management-Fullstack-App). GitHub connector not yet verified —
> `.factory/.env` with `GITHUB_TOKEN` was not present at init time; re-verify via `/factory:doctor`
> or on first `/factory:ba-sync` run.

## Docs (none configured)

### Page map (memory)
| Memory file | Docs page | synced-rev | Last sync |
|-------------|-----------|------------|-----------|
| domain-glossary.md | n/a (local-only) | 0 | never |
| stakeholders.md | n/a (local-only) | 0 | never |
| architecture.md | n/a (local-only) | 0 | never |
| decisions-integrations.md | n/a (local-only) | 0 | never |

### Artifacts map
| Artifact file | Docs page | Last sync |
|---------------|-----------|-----------|
| (added on publish) | n/a | never |

## PMO (GitHub Issues)

| Field | Value |
|-------|-------|
| Issue filter | TBD |

### Requirement map (FR-id ↔ issue)
| FR-id | GitHub issue # | Last sync |
|-------|----------------|-----------|
| (added on pmo-push) | TBD | never |
