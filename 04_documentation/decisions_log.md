# Decisions log

Newest last. Statuses: `accepted` · `open` · `superseded`.

## D1 — Repository per utility now, FlowERP monorepo later
- **Date:** 2026-11-26 · **Status:** accepted
- **Decision:** each utility and tool gets its own repository for isolated development and debugging. Later they are merged, with history, into the FlowERP monorepo (`utils/<name>`, `tools/<name>`). Asset accounting modules stay together in `assetrix`.
- **Why:** small repositories are easier to develop and debug one at a time; the final product is one system.
- **Consequence:** shared conventions and a `utility-template` repository are needed from the start, see [`ecosystem_overview.md`](ecosystem_overview.md).

## D2 — Utilities are libraries first
- **Date:** 2026-11-26 · **Status:** accepted
- **Decision:** a utility's core is a library; CLI and HTTP service are thin wrappers. Utilities do not depend on tools.
- **Why:** inside FlowERP, tools will call utilities in-process; standalone, the same code runs as a service.

## D3 — Python pilot first, Rust for the final version
- **Date:** 2026-11-26 · **Status:** accepted
- **Decision:** pilots in Python (FastAPI + Jinja + htmx + SQLite) to test usability quickly; final versions in Rust (Axum + Askama + htmx, sqlx). Web/PWA is the primary UI; desktop only as a Tauri wrapper over the same UI. The QR generator is written in Rust directly as the first learning project.
- **Why:** usability is learned fastest on a working pilot; Rust gives a light single binary, type-checked state logic and a long-term development path. The author is new to Rust.
- **Changes:** the specification's FastAPI + PostgreSQL stack becomes the pilot stack with SQLite; PostgreSQL remains an option for the final version.

## D4 — Assetrix scope extended; roles in the pilot
- **Date:** 2026-11-26 · **Status:** accepted
- **Decision:** Assetrix becomes an asset core (Asset incl. licences/certificates, Person, Department, Event) with modules: scanning, equipment movement, licence tracking, inventory. Login and roles are needed in the pilot. Departments start as a plain attribute.
- **Why:** these are the tools needed at work right now, and they share one data model.
- **Changes:** the MVP specification's single-user mode and out-of-scope list (inventory audits, procurement) need revision — Backlog task.

## D5 — Paid pro features
- **Date:** 2026-11-26 · **Status:** open
- **Decision so far:** paid pro features are planned; the model (open core, licence key, other) is not chosen.
- **Constraints until then:** keep extension points in the core (feature flags, pluggable modules); choose licences per repository before the first external contribution; check ownership of code written for work.
