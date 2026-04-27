# Assetrix — MVP Specification

## Goal

Build a **self-hosted PWA (web app)** for fast tracking of IT assets using:
- QR / barcode scanning
- event-based history
- bulk scan operations

Primary goal:
> perform any asset operation in 5–10 seconds from a smartphone

---

## Core Principles

### 1. Event Sourcing (STRICT)
- All changes = events
- NO update/delete of events
- History is immutable
- Current state = derived from events (with caching)

---

### 2. Speed over Completeness
Optimize for:
- minimal clicks
- mobile usage
- real-world IT workflow

Avoid:
- enterprise complexity
- over-engineering

---

## Data Model

### Device (Asset)

- id (UUID, primary key)
- asset_tag (string, NOT unique)
- name (string)
- type_id (FK)
- serial_number (string, required)
- created_at (timestamp)

Cached state:
- physical_condition (enum)
- operational_status (enum)
- last_event_id (FK)

---

### DeviceType

- id
- name (unique)

User-defined. No hierarchy.

---

### Event (immutable log)

- id
- device_id
- event_type
- note (optional text)
- meta (json)
- created_at
- created_by (nullable, but must exist in schema)

---

## Hard Rules

- NO editing events
- NO deleting events
- NO overwriting history

All corrections = new events

---

## Event Types (MVP)

- CREATED
- ASSIGNED
- RETURNED
- SENT_TO_REPAIR
- RETURNED_FROM_REPAIR
- WRITTEN_OFF

---

## Dual Status Model (CRITICAL)

Each device has TWO independent states:

### Physical Condition:
- virtual
- active
- maintenance
- damaged
- teardown
- scrap
- disposed

---

### Operational Status:
- available
- assigned
- reserved
- transit
- lost
- write_off

---

### Rules:
- exactly ONE value per category
- combined = full state

Example:
- assigned + damaged
- write_off + scrap
- write_off + disposed

---

## State Handling

- state derived from events
- state cached in Device
- manual “recalculate state” action must exist

---

## Asset Tag

- auto-generated via separate module/script
- must support:
  - unique assets
  - batch-based items (future)

---

## QR / Barcode

### QR:
- contains asset_tag

### Barcode:
- used for:
  - serial_number
  - vendor codes

---

### Unknown scan:
Show options:
- create device
- ignore
- add to pending list

---

## Scan Session (CORE FEATURE)

### Concept:
Universal bulk scanning system

---

### Flow:
1. Select mode
2. Scan devices
3. Apply action to all

---

### Modes:
- ASSIGN
- RETURN
- SEND_TO_REPAIR
- RETURN_FROM_REPAIR

---

### Behavior:

- duplicate scan:
  - unique devices → ignore
  - consumables → increase count (future)

---

### Context:
Defined BEFORE scanning

Example:
```json
{
  "assignee": "text"
}
```

---

## UX requirements:
- sound feedback
- visual feedback
- instant scan response
- remove scanned items
- one-button apply

---

## UI (Mobile-First)

### Core Screens:

1. Device List (main operational screen)
- search
- pagination
- sorting:
- - type
- - status
- - last event

2. Device Page
- QR code
- dual status display
- quick actions:
- - assign
- - send to repair
- - return
- - write off
- event history
- event filtering

3. Create Device

4. Scan Session 

---

### UX Rules:
- ≤ 3 clicks to action
- one-hand usage
- large touch controls
- minimal text

---

## Authentication (MVP)
- single user
- auto-login
- no logout

BUT: `created_by` field must exist

---

## Infrastructure

### Architecture:
- PWA (browser)
- HTTPS
- Backend API
- PostgreSQL

### Stack:
- Backend: FastAPI
- Database: PostgreSQL
- Reverse proxy: Caddy
- Deployment: Docker

### Startup:
`docker-compose up`

### Network:
- local network only (MVP)
- access via IP

---

## Scanning Support
Must support:
- smartphone camera (primary)
- laptop webcam
- USB barcode scanners (keyboard input)

## Business Logic
System must:
- prevent invalid transitions (e.g. written_off → assign)
- validate scenarios
- handle scan errors properly

## Reliability
- prevent duplicate events
- maintain audit trail
- no silent failures

## Performance Targets
- up to 10000 devices
- 2–3 scans/sec (barcode)
- 1 scan / 5 sec (camera)
- batch size: up to 50 devices

## Out of Scope (MVP)
DO NOT IMPLEMENT:
- multi-user system
- roles/permissions
- AD integration
- procurement
- warehouse model
- full inventory audits
- offline mode (future)

## Future Requirements (Design Constraints)
Must be extensible for:

multi-user
roles
integrations
inventory audits
consumables / batch tracking

## Definition of Done
System is complete when:
- opens on smartphone
- scans QR
- instantly finds device
- allows bulk operations
- logs events correctly
- shows accurate state

## Product Positioning
**This is NOT an enterprise system.**

This is: *a fast operational tool for IT technicians*

Focus on:
- speed
- simplicity
- real-world usage