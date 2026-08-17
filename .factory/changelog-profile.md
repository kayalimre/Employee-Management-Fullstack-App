# Changelog Profile

<!-- factory-version: 1.3.0 -->

> changelog-agent tuning (version source lives in project-profile -> Versioning).
> Last updated: 2026-08-17

## Format
- Style: **Keep a Changelog** (default). No `CHANGELOG.md` exists in the repository yet — the first run creates `CHANGELOG.md` at the project root with the standard header and an `## [Unreleased]` section.
- Sections: `Added`, `Changed`, `Deprecated`, `Removed`, `Fixed`, `Security` (omit empty ones).
- Grouping: by type, with the component named inline in the entry when it is not obvious (`(backend)`, `(frontend)`, `(infra)`, `(db)`) — this repo is a two-app fullstack system and readers need to know which side moved.
- Versioning is **per-app and unsynchronized** (root `package.json` 1.0.1, `frontend/package.json` 1.1.0, `backend/pom.xml` 0.0.1-SNAPSHOT), so a single repo-wide semantic version does not exist today. Head each release section with the root `package.json` version as the project-level marker and call out per-app version bumps in the entries. If the team later adopts a single release version, record that decision here.
- Source material: Conventional Commit subjects since the last release tag/section (`feat` → Added, `fix` → Fixed, `style`/`refactor`/`chore` → Changed, security-relevant fixes → Security). Merge commits (`Merge pull request #NN from ...`) are noise — read the PR's underlying commits instead, but keep the PR number as a reference where useful.
- Dates in `YYYY-MM-DD`. Base branch is `master`; the GitHub repo is `kayalimre/Employee-Management-Fullstack-App`.
- Release tag convention: TBD — no git tags are in use yet.
