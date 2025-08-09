# Pulse-Point — Project Kickstart (PRD + Tech Plan + Cursor TODOs)

## 0) Purpose & Goals

**One-liner:** Pulse-Point helps you track meetings, notes, and action items across contacts.

**Primary users:** managers, team leads, solo operators.

**Success metrics (MVP):**

* Create a contact/meeting/note/to-do in ≤ 10s from any screen.
* Find any note/to-do by keyword in ≤ 2 queries.
* Zero production errors on happy-path flows in first week.

**Non-goals (for MVP):** calendar/email sync, file attachments, multi-tenant roles/SSO, mobile app.

---

## 1) MVP Functional Requirements (with Acceptance Criteria)

### Contacts

* CRUD contacts (name, email, phone, company, title, freeform notes).
* Assign one or more **types** (e.g., Plumber, Client, Vendor).
* **AC:**

  * Unique type names; cannot create duplicates.
  * Contact list: filter by type, text search by name/company.

### Meetings

* CRUD meetings (title, description, start/end time, location/URL).
* Add attendees from contacts; roles optional (organizer/required/optional).
* Meeting view surfaces attached notes & to-dos.
* **AC:** `end_at` must be after `start_at` when provided; show inline editors for meeting-attached notes/to-dos.

### Notes

* Create **standalone** notes (random thoughts) and **meeting-attached** notes.
* Optional tags; markdown or rich text body.
* **AC:** Notes attached to a meeting display on that meeting page; tag filter works on notes list.

### To-dos / Action Items

* Standalone or meeting-attached.
* Fields: title (required), description, status (open/in\_progress/blocked/done), priority (low/med/high), due date, assignee (contact or “me”).
* **AC:** Default status `open`; list supports filter by assignee, status, due date.

### Cross-cutting

* Global search (contacts/meetings/notes/to-dos by keyword).
* Contact activity timeline (meetings, notes, open to-dos).
* Saved filters/views (local per-user in MVP).

---

## 2) Non-Functional Requirements

* **Performance:** P95 page nav < 2s; search results in < 300ms on 5k records.
* **Reliability:** health/readiness endpoints; seed data for demo.
* **Security:** server-side validation; audit fields; soft delete.
* **A11y:** keyboard navigable, focus states, ARIA on major components.
* **Observability:** structured logs with correlation IDs; basic metrics.

---

## 3) Domain Model (ERD Outline)

**IDs:** UUID (DB `uniqueidentifier` on SQL Server or `uuid` if Postgres). Below assumes **MS SQL Server**.

**contacts**

* id (uniqueidentifier, PK)
* first\_name, last\_name, email, phone, company, title
* notes (nullable)
* created\_at, updated\_at, deleted\_at (nullable)

**types**

* id (uniqueidentifier, PK)
* name (nvarchar(100), unique)
* color\_hex (nvarchar(7), nullable)
* created\_at

**contact\_types** (M\:N)

* id (uniqueidentifier, PK)
* contact\_id (FK → contacts)
* type\_id (FK → types)
* unique (contact\_id, type\_id)

**meetings**

* id (uniqueidentifier, PK)
* title, description (nullable)
* start\_at (datetime2), end\_at (datetime2, nullable)
* location (nullable), meeting\_url (nullable)
* created\_at, updated\_at, deleted\_at (nullable)

**meeting\_attendees** (M\:N)

* id (uniqueidentifier, PK)
* meeting\_id (FK → meetings)
* contact\_id (FK → contacts)
* role (nvarchar(30), nullable)
* unique (meeting\_id, contact\_id)

**notes**

* id (uniqueidentifier, PK)
* body (nvarchar(max))
* meeting\_id (nullable, FK → meetings)
* author\_contact\_id (nullable, FK → contacts)
* related\_contact\_id (nullable, FK → contacts)
* created\_at, updated\_at, deleted\_at (nullable)

**note\_tags**

* id (uniqueidentifier, PK)
* name (nvarchar(50), unique)

**note\_tag\_links**

* id (uniqueidentifier, PK)
* note\_id (FK → notes)
* note\_tag\_id (FK → note\_tags)
* unique (note\_id, note\_tag\_id)

**todos**

* id (uniqueidentifier, PK)
* title (nvarchar(200)), description (nvarchar(max), nullable)
* status (nvarchar(20)) — values: open, in\_progress, blocked, done
* priority (nvarchar(10)) — values: low, med, high
* due\_at (datetime2, nullable)
* assignee\_contact\_id (nullable, FK → contacts)
* meeting\_id (nullable, FK → meetings)
* related\_contact\_id (nullable, FK → contacts)
* created\_at, updated\_at, deleted\_at (nullable)

---

## 4) Initial SQL (SQL Server) — Migrations v1

> Use TypeORM migrations; below are table shapes for the first migration.

```sql
CREATE TABLE types (
  id UNIQUEIDENTIFIER NOT NULL DEFAULT NEWID() PRIMARY KEY,
  name NVARCHAR(100) NOT NULL UNIQUE,
  color_hex NVARCHAR(7) NULL,
  created_at DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME()
);

CREATE TABLE contacts (
  id UNIQUEIDENTIFIER NOT NULL DEFAULT NEWID() PRIMARY KEY,
  first_name NVARCHAR(100) NOT NULL,
  last_name NVARCHAR(100) NOT NULL,
  email NVARCHAR(255) NULL,
  phone NVARCHAR(50) NULL,
  company NVARCHAR(150) NULL,
  title NVARCHAR(150) NULL,
  notes NVARCHAR(MAX) NULL,
  created_at DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
  updated_at DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
  deleted_at DATETIME2 NULL
);

CREATE TABLE contact_types (
  id UNIQUEIDENTIFIER NOT NULL DEFAULT NEWID() PRIMARY KEY,
  contact_id UNIQUEIDENTIFIER NOT NULL,
  type_id UNIQUEIDENTIFIER NOT NULL,
  CONSTRAINT FK_contact_types_contact FOREIGN KEY (contact_id) REFERENCES contacts(id),
  CONSTRAINT FK_contact_types_type FOREIGN KEY (type_id) REFERENCES types(id),
  CONSTRAINT UQ_contact_type UNIQUE (contact_id, type_id)
);

CREATE TABLE meetings (
  id UNIQUEIDENTIFIER NOT NULL DEFAULT NEWID() PRIMARY KEY,
  title NVARCHAR(200) NOT NULL,
  description NVARCHAR(MAX) NULL,
  start_at DATETIME2 NOT NULL,
  end_at DATETIME2 NULL,
  location NVARCHAR(200) NULL,
  meeting_url NVARCHAR(300) NULL,
  created_at DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
  updated_at DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
  deleted_at DATETIME2 NULL
);

CREATE TABLE meeting_attendees (
  id UNIQUEIDENTIFIER NOT NULL DEFAULT NEWID() PRIMARY KEY,
  meeting_id UNIQUEIDENTIFIER NOT NULL,
  contact_id UNIQUEIDENTIFIER NOT NULL,
  role NVARCHAR(30) NULL,
  CONSTRAINT FK_att_meeting FOREIGN KEY (meeting_id) REFERENCES meetings(id),
  CONSTRAINT FK_att_contact FOREIGN KEY (contact_id) REFERENCES contacts(id),
  CONSTRAINT UQ_meeting_contact UNIQUE (meeting_id, contact_id)
);

CREATE TABLE notes (
  id UNIQUEIDENTIFIER NOT NULL DEFAULT NEWID() PRIMARY KEY,
  body NVARCHAR(MAX) NOT NULL,
  meeting_id UNIQUEIDENTIFIER NULL,
  author_contact_id UNIQUEIDENTIFIER NULL,
  related_contact_id UNIQUEIDENTIFIER NULL,
  created_at DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
  updated_at DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
  deleted_at DATETIME2 NULL,
  CONSTRAINT FK_note_meeting FOREIGN KEY (meeting_id) REFERENCES meetings(id),
  CONSTRAINT FK_note_author FOREIGN KEY (author_contact_id) REFERENCES contacts(id),
  CONSTRAINT FK_note_related_contact FOREIGN KEY (related_contact_id) REFERENCES contacts(id)
);

CREATE TABLE note_tags (
  id UNIQUEIDENTIFIER NOT NULL DEFAULT NEWID() PRIMARY KEY,
  name NVARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE note_tag_links (
  id UNIQUEIDENTIFIER NOT NULL DEFAULT NEWID() PRIMARY KEY,
  note_id UNIQUEIDENTIFIER NOT NULL,
  note_tag_id UNIQUEIDENTIFIER NOT NULL,
  CONSTRAINT FK_ntl_note FOREIGN KEY (note_id) REFERENCES notes(id),
  CONSTRAINT FK_ntl_tag FOREIGN KEY (note_tag_id) REFERENCES note_tags(id),
  CONSTRAINT UQ_note_tag UNIQUE (note_id, note_tag_id)
);

CREATE TABLE todos (
  id UNIQUEIDENTIFIER NOT NULL DEFAULT NEWID() PRIMARY KEY,
  title NVARCHAR(200) NOT NULL,
  description NVARCHAR(MAX) NULL,
  status NVARCHAR(20) NOT NULL DEFAULT 'open',
  priority NVARCHAR(10) NOT NULL DEFAULT 'med',
  due_at DATETIME2 NULL,
  assignee_contact_id UNIQUEIDENTIFIER NULL,
  meeting_id UNIQUEIDENTIFIER NULL,
  related_contact_id UNIQUEIDENTIFIER NULL,
  created_at DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
  updated_at DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
  deleted_at DATETIME2 NULL,
  CONSTRAINT FK_todo_assignee FOREIGN KEY (assignee_contact_id) REFERENCES contacts(id),
  CONSTRAINT FK_todo_meeting FOREIGN KEY (meeting_id) REFERENCES meetings(id),
  CONSTRAINT FK_todo_related_contact FOREIGN KEY (related_contact_id) REFERENCES contacts(id)
);
```

---

## 5) API Surface (NestJS + TypeORM, REST v1)

**Contacts & Types**

* `POST /v1/contacts` (create), `GET /v1/contacts?search=&type=` (list+filter), `GET /v1/contacts/:id` (detail)
* `PATCH /v1/contacts/:id`, `DELETE /v1/contacts/:id`
* `POST /v1/types`, `GET /v1/types`
* `POST /v1/contacts/:id/types/:typeId`, `DELETE /v1/contacts/:id/types/:typeId`
* `GET /v1/contacts/:id/activity` → \[meetings|notes|todos]

**Meetings**

* `POST /v1/meetings`, `GET /v1/meetings?from=&to=&attendeeId=`
* `GET /v1/meetings/:id`, `PATCH /v1/meetings/:id`, `DELETE /v1/meetings/:id`
* `POST /v1/meetings/:id/attendees` (bulk add), `DELETE /v1/meetings/:id/attendees/:contactId`
* `GET /v1/meetings/:id/summary` → attendees, notes, todos

**Notes**

* `POST /v1/notes` (supports optional `meetingId`, `relatedContactId`)
* `GET /v1/notes?meetingId=&contactId=&tag=&q=`
* `PATCH /v1/notes/:id`, `DELETE /v1/notes/:id`
* `POST /v1/note-tags`, `GET /v1/note-tags`, `POST /v1/notes/:id/tags/:tagId`, `DELETE ...`

**To-dos**

* `POST /v1/todos`
* `GET /v1/todos?meetingId=&assigneeId=&status=&dueBefore=&q=`
* `PATCH /v1/todos/:id`, `DELETE /v1/todos/:id`

**Search**

* `GET /v1/search?q=` → union of top matches across entities.

**System**

* `GET /health` (liveness), `GET /ready` (db check)

---

## 6) UX Flows & Screens (low-fi outline)

* **Global nav:** Dashboard • Contacts • Meetings • Notes • To-dos.
* **Quick create:** floating "+" → Contact/Meeting/Note/To-do.
* **Contacts list:** search + type filter → contact detail (activity timeline + quick Add Note/To-do).
* **Meetings list/calendar:** meeting detail shows attendees + inlined notes/to-dos.
* **Notes:** feed + tag filter; inline editor.
* **To-dos:** list + Kanban (Open / In Progress / Blocked / Done).

---

## 7) Tech Decisions (ADR Summaries)

* **Stack:** Nx monorepo; React (web), NestJS (api); TypeORM; **SQL Server**.
* **Styling/UI:** Tailwind + headless UI primitives; optional shadcn for speed.
* **State/data:** RTK Query for API data; Zod on client for request/response parsing.
* **Auth:** none for MVP (single-user). Add Keycloak later.
* **IDs:** UUID (DB `uniqueidentifier`); soft delete via `deleted_at`.
* **Logging:** pino-like JSON logs (api); client error boundary + Sentry-ready hook.

---

## 8) Dev Environment

* **Docker Compose (dev):** sqlserver (2019+), azurite not needed; admin: SQL Server Web Tools in container or use local SSMS.
* **Ports:** api `http://localhost:3333`, web `http://localhost:4200` (or 5173 if Vite).
* **Env:** `.env` for DB creds; separate `.env.local` for web.

---

## 9) Seed Data (MVP)

* Types: Client, Vendor, Plumber, Partner.
* Contacts: a few people across types.
* One meeting with two attendees; two attached notes; three to-dos (one overdue).

---

## 10) Milestones

* **M1 (Week 1–2):** Contacts & Types (CRUD + filters) + database + seed.
* **M2 (Week 2–3):** Meetings + attendees + attached notes/to-dos.
* **M3 (Week 3–4):** Standalone notes/to-dos + global search + saved views.
* **M4 (Week 4–5):** Polish, a11y pass, empty states, packaging.

---

## 11) Cursor TODO List (give this section to Cursor)

### A) Repo & Tooling

* [ ] Create Nx monorepo: apps `web` (React + Vite), `api` (NestJS), libs `models`, `data-access`, `ui`.
* [ ] Configure ESLint + Prettier; set strict TS in all projects.
* [ ] Add Husky + lint-staged: run `eslint --fix`, `type-check`, and unit tests on pre-commit.
* [ ] Set up GitHub Actions: `lint → test → build → docker` for both web and api.

### B) API (NestJS + TypeORM + SQL Server)

* [ ] Add TypeORM with SQL Server driver; load connection from `.env`.
* [ ] Implement entities for: Contact, Type, ContactType, Meeting, MeetingAttendee, Note, NoteTag, NoteTagLink, Todo.
* [ ] Generate **Migration #1** with all tables above; run against `sqlserver` container.
* [ ] Implement DTOs + validation (class-validator) for create/update endpoints.
* [ ] Implement controllers/services for:

  * [ ] `/v1/types` (list/create)
  * [ ] `/v1/contacts` (CRUD, search, type assign/remove)
  * [ ] `/v1/meetings` (CRUD, list with range + attendee filter; attendees add/remove)
  * [ ] `/v1/notes` (CRUD, list with filters; tag CRUD + link/unlink)
  * [ ] `/v1/todos` (CRUD, list with filters)
  * [ ] `/v1/search` (simple keyword union across entities)
  * [ ] `/health`, `/ready`
* [ ] Add soft-delete behavior (set `deleted_at` instead of hard delete); add scopes to exclude deleted by default.
* [ ] Add request logging middleware with correlation ID; log endpoint, duration, status.
* [ ] Seed script: initial types, a few contacts, a meeting with attached items.

### C) Web (React + Vite + Tailwind + RTK Query)

* [ ] Initialize Tailwind; set base theme + dark mode class strategy.
* [ ] Set up RTK Query base API with `fetchBaseQuery` using env var.
* [ ] Define shared types (from `models` lib) and Zod schemas for API responses.
* [ ] Layout: left nav (Dashboard, Contacts, Meetings, Notes, To-dos); top bar with global search.
* [ ] **Contacts**

  * [ ] List view with search + type filters; pagination.
  * [ ] Detail view: profile panel + activity timeline (meetings, notes, open to-dos).
  * [ ] Create/Edit drawer.
* [ ] **Meetings**

  * [ ] List (table) + week calendar (basic) — can stub calendar with list first.
  * [ ] Detail: attendees, agenda; inline sections: notes and to-dos filtered by `meetingId`.
  * [ ] Add attendees via multi-select of contacts.
* [ ] **Notes**

  * [ ] Feed with tag filter + search; editor supports markdown (simple toolbar).
  * [ ] Create note (optionally select meeting and/or related contact).
* [ ] **To-dos**

  * [ ] List + Kanban board (Open/In Progress/Blocked/Done).
  * [ ] Create to-do (optional meeting/contact association); edit status/priority/due.
* [ ] **Global search**

  * [ ] Search box in top bar; show grouped results (contacts/meetings/notes/to-dos).
* [ ] **Saved views** (local): store JSON in `localStorage` for common filters.
* [ ] Error boundary + empty states + loading skeletons.

### D) Dev & Ops

* [ ] Docker Compose with services: `sqlserver`, `api`, `web`.
* [ ] Healthchecks for `api`; wait-for-it in `web` to start after `api` is ready (optional).
* [ ] `.env.example` for api and web; document setup steps.
* [ ] Add Playwright e2e smoke: create contact → create meeting → attach note/to-do → verify on meeting page.

### E) Nice-to-haves (post-MVP toggles)

* [ ] Mention linking in notes (`@ContactName`) → auto-relate to contact.
* [ ] To-do quick-add suggestions after meeting ends (extract action items later with AI).
* [ ] Keyboard shortcuts: `N` (new note), `T` (new to-do), `C` (new contact), `M` (new meeting).

---

## 12) Issue Backlog (import-friendly)

* [ ] PRD + ADRs committed to repo
* [ ] Nx monorepo scaffolding
* [ ] Tailwind setup + base styles
* [ ] TypeORM + SQL Server wiring
* [ ] Migration #1 tables
* [ ] Seed script
* [ ] Contacts API endpoints + tests
* [ ] Types API endpoints + tests
* [ ] Meetings API endpoints + tests
* [ ] Notes API endpoints + tests
* [ ] To-dos API endpoints + tests
* [ ] Search endpoint + tests
* [ ] Web: Contacts list/detail
* [ ] Web: Meetings list/detail (with inline notes/to-dos)
* [ ] Web: Notes feed/editor
* [ ] Web: To-dos list/Kanban
* [ ] Global search UI
* [ ] Contact activity timeline
* [ ] Empty states + error boundary
* [ ] Playwright e2e smoke
* [ ] Docker Compose + docs

---

## 13) Open Questions (to confirm as you go)

* Keep SQL Server for prod, or dev on SQL Server and allow optional Postgres adapter?
* Markdown vs. rich text editor preference?
* Calendar view: list-only for MVP or add a basic week grid now?
* Single-user (no auth) MVP acceptable, or enable Keycloak early?
