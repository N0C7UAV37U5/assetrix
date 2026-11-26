# Ecosystem overview

Status as of 2026-11-26. Decisions behind this overview: [`decisions_log.md`](decisions_log.md).

## Hierarchy

```
Mycorium (umbrella ecosystem)
└── FlowERP (product, future monorepo)
    └── modules / tools: Assetrix, Paginator, ...
        └── utilities: QR/barcode generator, PDF converter, file sanitizer, ...
```

- **Utilities** are small, stateless building blocks: data or a file in, a file out. They know nothing about tools.
- **Tools** combine utilities with their own domain logic. Example: Assetrix uses the QR generator and the PDF converter to print asset labels.
- **FlowERP** assembles tools as modules into one system.

## Repositories

- **Now:** one repository per utility or tool, for isolated development and debugging.
- **Later:** everything is merged into the FlowERP monorepo, with history (`git subtree` or `git filter-repo`).
- Target paths in the monorepo: `utils/<name>`, `tools/<name>`.
- All asset accounting modules (scanning, equipment movement, licences, inventory) live in one repository, `assetrix`: they share one database and one event log.

## Domains and URLs

| Domain | Role |
|---|---|
| `flowerp.app` | Running product instances: `<tenant>.flowerp.app/<module>` |
| `flowerp.org` | Product site, documentation, community |
| `mycorium.org` | Ecosystem umbrella; `*.lab.mycorium.org` for the homelab and test instances |

Tenant in the host name, module in the path; labels use the permanent `/a/<code>` path at the tenant root. Decisions D17, D18; setup in [`domains_setup.md`](domains_setup.md).

## Common conventions (from day one)

So that the later merge is mechanical rather than a manual cleanup:

| Area | Convention |
|---|---|
| Structure | Same folder layout in every repository, created from a `utility-template` repository |
| Code shape | Library first: core logic is a library; CLI and HTTP service are thin wrappers. FlowERP can then call utilities in-process |
| Rust | Plain Cargo crates (workspace-compatible), shared `rustfmt` / `clippy` config |
| Python pilots | `uv` + `ruff` |
| HTTP API | OpenAPI contract in the repository; one error format; `/health` endpoint; versions via git tags, breaking change = new major version |
| Long operations | Background jobs: accept, return a job id, fetch the result later |
| Config and logs | Environment variables; JSON logs |
| CI | One shared CI template |

Later shared repositories (only when there is more than one consumer): `mycorium-ui` (CSS tokens and components), `mycorium-deploy` (`docker-compose` + Caddyfile for the whole ecosystem).

## Technology tracks

| Track | Purpose | Stack |
|---|---|---|
| Pilot | Ship quickly, test usability in real work | Python: FastAPI + Jinja + htmx + SQLite |
| Final | Lightweight, fast, single binary | Rust: Axum + Askama + htmx, sqlx |

- What carries over from pilot to final: database schema, event model, API contract, HTML/CSS and UX. The server code is rewritten.
- Web/PWA is the primary UI. Desktop, where needed, is a Tauri wrapper around the same web UI.
- Exception: the QR generator is written in Rust directly, as the first Rust learning project.

## Assetrix scope

Core entities: **Asset** (devices, licences, certificates) with external **identifiers**, **Person**, **Department**, **Organization**, **Location** (tree), **Warehouse** (virtual or physical), **Event** (immutable log). Pilot specification: [`assetrix_spec_v2.md`](../01_sources/comp/assetrix_spec_v2.md).

| Module | Adds |
|---|---|
| Scanning | Scan sessions, batch operations (the original MVP) |
| Equipment movement | Onboarding kit templates, offboarding list, temporary replacement during repair (one operation, several events), moves between departments |
| Licences and certificates | Expiry dates, seats, assignment to people or assets, reminders and a dashboard |
| Inventory | Reconciliation sessions (expected vs found), result report, one-click follow-up events; network scan as a separate collector |

- Login and roles come in pilot stage P2; P1 is single-user (D12).
- Departments start as a plain attribute; reconciliation with 1C/ERP is undecided.

## Utilities

| Utility | Approach |
|---|---|
| QR / barcode generator | Rust crates `qrcode`, `barcoders`, `rxing`; output SVG / PNG / PDF |
| PDF converter | Wrap existing engines, do not write one: WeasyPrint (HTML + CSS paged media), Markdown → HTML → PDF, LibreOffice headless for office formats; Gotenberg as a ready-made option |
| File sanitizer | Rebuild files rather than inspect them: re-encode images, remux video without metadata, strip JS and attachments from PDF or rasterise pages. Run in a sandbox (container without network, resource limits). Reference: Dangerzone |
| Paginator | Own block-based document model (JSON); ProseMirror for text editing inside blocks; rendering via Paged.js / WeasyPrint / Chromium, or Typst; EPUB export |

## Order

1. QR generator (Rust)
2. Asset core + Assetrix pilot (Python)
3. PDF converter
4. Equipment movement → licences → inventory; sanitizer as soon as file uploads appear
5. Paginator
