# Assetrix

Self-hosted PWA for fast tracking of IT assets using QR / barcode scanning, event-based history and bulk scan operations. Part of the Mycorium ecosystem.

## Project summary

Stage: design. No application code yet.

Planned stack: React PWA, FastAPI (Python), PostgreSQL, Caddy, Docker.

Specification (in Russian): [`01_sources/comp/assetrix_mvp_spec.md`](01_sources/comp/assetrix_mvp_spec.md). Additional details: [`01_sources/raw/mvp_info.md`](01_sources/raw/mvp_info.md); the specification takes precedence.

## Project location & access

- Repository: <https://github.com/N0C7UAV37U5/assetrix>
- License: Apache 2.0, see [`LICENSE`](LICENSE)

## Project structure

| Folder | Purpose |
|--------|---------|
| `01_sources/raw/` | Inputs as received: notes, drafts, loose ideas |
| `01_sources/comp/` | Compiled, continuously updated specifications and requirements |
| `02_workflow/<feature>/` | Working area, one subfolder per feature or task |
| `03_result/` | Current result (if applicable) |
| `04_documentation/` | Manuals, descriptions, changelog, decisions log, references |
| `05_assets/` | Mockups, wireframes, images, media |
| `06_archive/` | Deprecated items (moved here instead of deleting) |
| `README.md` | This file: structure map |
| `Backlog.md` | Living project backlog |
| `CLAUDE.md` | Project context for Claude Code |

## Naming convention

No spaces, lowercase, `_`/`-` separators, no trailing dots.
Avoid characters `" * : < > ? / \ |`.
