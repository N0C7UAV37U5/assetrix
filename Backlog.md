# Backlog — Assetrix

Living project backlog. Single source of truth for tasks and features.
Specification: [`01_sources/comp/assetrix_spec_v2.md`](01_sources/comp/assetrix_spec_v2.md). Decisions: [`04_documentation/decisions_log.md`](04_documentation/decisions_log.md).

## Status legend

`new` · `in progress` · `blocked` · `review` · `done` · `archived`

## Tasks

| ID | Feature / task | Priority | Status | Notes |
|----|----------------|----------|--------|-------|
| 1 | Decide Python version and dependency manager | High | done | Python 3.12 + `uv` + `ruff` + `pytest` (spec v2, section 3) |
| 2 | Choose a browser QR / barcode library | High | done | `BarcodeDetector` + `barcode-detector` polyfill (zxing-wasm) |
| 3 | Define the asset code format | High | done | `asset_code`, Crockford Base32 7+1 check (D8); provisional, verify in the pilot |
| 4 | Set up the dev environment | Medium | new | 4+ homelab machines via Ansible, state unknown. Not blocking: P1 needs one Linux host with Docker |
| 5 | Align the operation time target | Medium | done | Single operation 3–5 s; batch scan response ≤ 1 s |
| 6 | Create the pilot project skeleton | High | new | FastAPI + Jinja + htmx + SQLite, SQL migrations compatible with `sqlx migrate`, Docker Compose + Caddy (with `caddy-dns/porkbun`); runs under a base path `/assetrix/` from day one (D17, spec v2 section 3.1) |
| 7 | Fix `package.json` | Low | new | `main` points to a missing `index.js`; no JS build is planned — probably move to `06_archive/` |
| 8 | Revise the MVP specification | High | done | `assetrix_spec_v2.md`; v1 in `06_archive/` |
| 9 | Define the state transition table | High | done | Spec v2, section 5.2; refine during the pilot |
| 10 | Define the Person and Department model | High | done | Spec v2, section 4.4; persons from Vikarina |
| 11 | Create the `utility-template` repository | Medium | new | Shared layout, conventions and CI for all utilities (decision D1) |
| 12 | Start the QR generator repository (Rust) | Medium | new | First Rust learning project; library + CLI, then HTTP (decision D2). Parallel, does not block the pilot |
| 13 | Choose the licence and open-core strategy | Medium | new | Per repository, before the first external contribution (D5, D10) |
| 14 | Put the code ownership arrangement in writing | High | new | IP stays with the author, employer gets a perpetual full-access licence (D6) |
| 15 | Agree on crate / package naming | Low | new | Mycorium / FlowERP prefix |
| 16 | Collect and categorise asset types | High | new | Gather asset types across the company, categorise and parametrise them; derive types, categories, required and optional fields. Until then: fixed minimum fields + `attributes` JSON |
| 17 | Buy a domain and set up HTTPS | High | in progress | Bought: `flowerp.app`, `flowerp.org` (+ owned `mycorium.org`); roles D18. Remaining: task 30, then Caddy on the servers |
| 18 | Choose and buy a label printer | Medium | new | Thermal transfer, resin + polyester, 300 dpi, network, ZPL (D11); fixes the label size |
| 19 | Prepare the Excel import mapping | High | done | `01_sources/comp/import_mapping_spare.md`; `Summary` sheet fully mapped, import all rows |
| 20 | Write the Vikarina employee export query | Medium | new | MS SQL → CSV: external id, full name, department, email, active. No AD login in Vikarina: `login` = email local part, fallback ASCII transliteration `name.surname` |
| 21 | Implement pilot P1.0 | High | new | Spec v2, sections 4–9; DoD in section 11 |
| 22 | Implement inventory P1.1 | High | new | Spec v2, section 6.4 |
| 23 | Prompt for assets on a person's department change | Medium | new | P1.x; spec v2, section 5.4 |
| 24 | Keep real data out of git | High | done | `.gitignore` for Excel/CSV/DB/backup files (D15); `Spare.xlsx` stays local |
| 25 | Map the `Office` licence sheet for P3 | Low | new | Structure in `import_mapping_spare.md`, section 2; binding by inventory number or host name; report licences bound to PCs missing from `Summary` |
| 26 | Protect licence keys before P3 | Medium | new | Encryption at rest or hidden display + reveal audit (D14) |
| 27 | Discuss disassembly of donor PCs into parts | Medium | new | One accounting item "computer" becomes many part items: events, identifiers of parts, how accounting reflects it (write-off of the PC, recognition of parts, value split). First candidates: imported PCs with `ad_disabled` |
| 28 | Investigate Office keys with activation errors | Low | new | Status meanings clarified (`import_mapping_spare.md`, section 2); the cause of activation errors is unknown |
| 29 | Design transfer of assets to another legal entity of the group | Low | new | Disposal scenario for hardware and licences (e.g. PCs with their Office keys handed over); write-off with reason + receiving organization |
| 30 | Porkbun: API key, API access, DNS records | High | in progress | Done 2026-11-27: `flowerp.app` API key + API Access; work server reaches Porkbun and Let's Encrypt, ports 80/443 free. Remaining: A record `ut`, `mycorium.org`. Originally: Steps 1–4 in `04_documentation/domains_setup.md`: API key to a password manager; API Access on for `flowerp.app` and `mycorium.org`; A records `ut.flowerp.app`, `*.lab.mycorium.org`; check DNS from a phone on the work Wi-Fi |
| 31 | Check the FlowERP name for conflicts | Low | new | Existing products and trademarks (EUIPO / TMview) before public launch |

## Ideas / later

- **P2:** login and roles (permission sets), transfer with recipient confirmation (`transit`), reservations, onboarding kits, offboarding list, temporary replacement during repair.
- **P3:** licences as assets — assignment to a person or binding to an asset; on asset return auto-release or a suspension queue for the admin (e.g. Claude Team seat).
- **Components and assemblies:** asset inside asset, `INSTALLED` / `REMOVED` events; rebuilt PC keeps its old number with a clean history.
- **Pro:** warehouse addressing (zones, aisles, racks, levels, slots), distribution, storage conditions (D10).
- **Labels:** server-side PDF via the PDF converter utility; direct network printing (ZPL).
- **Integrations:** live Vikarina sync, AD, Axapta.
- **FlowERP:** on staffing changes (transfer, dismissal) always prompt a decision on the assets held by the person.
