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
- **Update 2026-11-26:** first known boundary, see D10.

## D6 — Lifetime full-access key for the employer
- **Date:** 2026-11-26 · **Status:** accepted
- **Decision:** the author develops in free time at work (agreed with the employer) and at home. The pilot is tested on real work tasks. The employer receives a perpetual full-access licence key, in recognition of the work time used.
- **Why:** fairness, and the project is meant to become an open, transparent alternative to the company's Axapta 3 system (via FlowERP).
- **Action:** get the arrangement in writing (IP stays with the author, the company gets a perpetual licence) — Backlog task 14.

## D7 — HTTPS through an own domain and DNS-01; no VPS yet
- **Date:** 2026-11-26 · **Status:** accepted
- **Decision:** buy a short domain at Porkbun. Caddy (built with `caddy-dns/porkbun`) gets a wildcard certificate `*.lab.<domain>` through the DNS-01 challenge. Public A records point to private IPs of the homelab and work servers. A VPS is not needed for the pilot.
- **Why:** the company DNS is not under our control; the camera API needs a secure context; a wildcard certificate keeps host names out of Certificate Transparency logs.
- **Risk:** some DNS resolvers block public names that resolve to private IPs (DNS rebinding protection). Check once from a phone on the work Wi-Fi; fallback is a local CA installed on the phones.
- **Later:** a VPS for a public demo, a VPN hub, CI or off-site backups. Work data (employee names, assignments) stays on the work server.
- **Interim options until the domain is bought (2026-11-26):**
  1. *Development:* `http://localhost` is a secure context, so the laptop webcam works without certificates; an Android phone reaches the dev machine over USB with `adb reverse tcp:8000 tcp:8000`.
  2. *Phone trials:* free DNS with an API — deSEC.io (`caddy-dns/desec`), DuckDNS (`caddy-dns/duckdns`, less reliable) — gives a real wildcard certificate through DNS-01 with the same setup as Porkbun; moving to the bought domain only swaps the Caddy module and the name.
  3. *Fallback:* `sslip.io` / `nip.io` names (e.g. `192-168-1-10.sslip.io`) + Caddy `tls internal`, with Caddy's root certificate installed on test phones. No public certificate is possible there: their DNS is not ours (no DNS-01) and a private IP is unreachable for HTTP-01.
- **Status of option 2 (2026-11-26):** deSEC has suspended `dedyn.io` registrations until further notice (abuse wave); a deSEC account is useful only for hosting DNS of an own domain. DuckDNS signs in only through Google, GitHub, X/Twitter or Reddit and gives single-level names (`<name>.duckdns.org`; deeper names resolve to the same IP). The author looks for a domain on their own.
- **Rebinding check (2026-11-26):** the DNS on the author's work machine resolves `*.sslip.io` / `*.nip.io` to private IPs, so it does not block rebinding. Still to check from a phone on the work Wi-Fi.

## D8 — Asset identification: own code plus external identifiers
- **Date:** 2026-11-26 · **Status:** accepted
- **Decision:** every asset gets an own `asset_code` (Crockford Base32, 7 random characters plus a Luhn mod 32 check character, shown as `XXXX-XXXX`). The QR code encodes a URL with it. All other codes (`invent_num`, `service_tag`, `serial_num`, `license_key`, `vin_code`, …) live in an `identifiers` table with user-defined schemes. A scan searches both.
- **Why:** real identifiers have unrelated, changing formats and many assets have none. Existing labels and vendor barcodes keep working without relabelling.
- **Details:** [`assetrix_spec_v2.md`](../01_sources/comp/assetrix_spec_v2.md), sections 4.2–4.3.

## D9 — Holder, balance, warehouse and location are separate
- **Date:** 2026-11-26 · **Status:** accepted
- **Decision:** the holder (person or department) answers "who is responsible"; the balance department answers "whose books"; the warehouse (optional) and the location (tree) answer "where it is stored / where it physically is". A warehouse is never a holder.
- **Why:** a laptop follows its user, a desktop stays in the room; inventory by room and by department need different axes.
- **Default:** nothing moves implicitly — on assignment and on a person's department change the system asks, default "no change".

## D10 — Warehouses: virtual free, addressing pro
- **Date:** 2026-11-26 · **Status:** accepted
- **Decision:** warehouses are optional and many-to-many with departments; every new department gets a default virtual warehouse. Free: virtual and physical warehouses without addressing. Pro: addressing (zones, aisles, racks, levels, slots), distribution, storage conditions (humidity, temperature, shelf life, periodic handling).
- **Why:** first concrete free/pro boundary for D5.

## D11 — Labels printed from the browser in the pilot
- **Date:** 2026-11-26 · **Status:** accepted
- **Decision:** the pilot renders labels as an HTML page with CSS `@page` and uses the browser print dialog (printer or PDF): single label and A4 sheet. Server-side PDF comes with the PDF converter utility; direct network printing (ZPL) later. The label shows the QR code, `asset_code` large, `invent_num` small; no company name.
- **Printer criteria:** thermal transfer with resin ribbon on polyester labels, 300 dpi, network interface, ZPL.

## D12 — Pilot P1 is single-user; roles in P2
- **Date:** 2026-11-26 · **Status:** accepted
- **Decision:** P1 runs as one system user behind Caddy `basic_auth`. Login and roles come in P2, as permission sets with working names `admin`, `it`, `manager`/`dept_head`, `accountant`, `storekeeper`/`logistics`, `employee`.
- **Changes:** refines D4, which put roles into the pilot from the start.

## D13 — Data sources for the pilot
- **Date:** 2026-11-26 · **Status:** accepted
- **Decision:** P1 imports IT assets from the IT department's Excel file and employees from a CSV export of Vikarina (MS SQL). Axapta is the accounting master for the future system (purchase history, balance, value changes, later production goods), but is not imported in P1.
- **Update 2026-11-26:** the Excel file is `Spare.xlsx` (sheets `Summary` — hardware, `Office` — Office licences). Mapping: [`import_mapping_spare.md`](../01_sources/comp/import_mapping_spare.md).

## D14 — No licence keys in the pilot P1
- **Date:** 2026-11-26 · **Status:** accepted
- **Decision:** Windows and Office keys from the Excel file are not imported in P1; only versions go into asset attributes. Keys come with P3, where licences become assets with the `license_key` scheme (sensitive).
- **Why:** P1 is protected only by `basic_auth`; keys are secrets and are not needed for issuing and returning hardware. Before P3: encryption at rest or at least hidden-by-default display and an audit of who revealed a key.

## D15 — Real source data never goes into git
- **Date:** 2026-11-26 · **Status:** accepted
- **Decision:** files with real data (Excel and CSV exports, database files, backups) are excluded by `.gitignore` and stay only on local and work machines. Documentation describes only their structure: headers, value shapes, counts.
- **Why:** the GitHub repository is public; the data contains employee names, logins and licence keys.

## D16 — Identifier uniqueness warns, does not block
- **Date:** 2026-11-26 · **Status:** accepted
- **Decision:** a scheme marked `is_unique` shows a warning on a duplicate value instead of rejecting it; a scan that finds several assets offers a choice.
- **Why:** the real data already has duplicate inventory numbers; blocking would stop the import and hide the problem instead of showing it.
