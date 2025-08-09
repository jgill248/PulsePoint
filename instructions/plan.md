# Pulse-Point — Software Development Plan

**Stack:** Nx Workspace (already exists) • React • TailwindCSS • MUI • Mantine React Table • TypeScript • Node.js API (Nest.js or Express) • PostgreSQL

> Goal: Build a professional, enterprise-style meeting/notes/todo/contact management system.

---

## 1. Project Prep

* [ ] Confirm app `pulse-point` exists in Nx workspace.
* [ ] Install Tailwind, MUI, Mantine React Table, React Router.
* [ ] Configure ESLint, Prettier, Husky (pre-commit lint + test).
* [ ] Add `.env` handling via `dotenv` or Nx environment config.

---

## 2. Core Features & Entities

* **Contacts** — name, contact info, tags/types (e.g., "Plumber").
* **Meetings** — date/time, attendees (contacts), location, agenda.
* **Notes** — linked to meetings or standalone.
* **Todos** — linked to meetings or standalone; status, due date.

---

## 3. Nx-First Workflow Rules

* Always use Nx CLI to generate code:

  ```bash
  nx g @nx/react:component component-name --project=pulse-point --directory=components
  nx g @nx/react:lib lib-name
  nx g @nx/js:library data-access-entity
  nx g @nx/node:lib api-entity
  ```
* Group code into `feature-*`, `ui-*`, `data-access-*` libraries.
* Keep business logic in `data-access` libs; components consume via hooks/services.

---

## 4. Development Steps

### Step 1 — Data Models & API

1. Create shared TypeScript interfaces for Contact, Meeting, Note, Todo in `libs/data-models`.
2. Create Nest.js API (or extend existing) with CRUD endpoints for each entity.
3. Implement PostgreSQL entities + migrations (e.g., via Prisma or TypeORM).
4. Add seed script for sample data.

### Step 2 — Data Access Layer

1. Generate `data-access-contacts`, `data-access-meetings`, etc.
2. Implement RTK Query or React Query hooks for API calls.
3. Add optimistic updates where applicable.

### Step 3 — UI Components

1. **Layout Shell:** header, sidebar, content area.
2. **Contacts:** list (Mantine React Table), detail, create/edit drawer.
3. **Meetings:** list, calendar view, detail (notes & todos tabs).
4. **Notes:** list, rich-text editor, linked/standalone toggle.
5. **Todos:** list, Kanban view, linked/standalone toggle.

### Step 4 — Routing & Navigation

1. Configure React Router pages: `/contacts`, `/meetings`, `/notes`, `/todos`.
2. Add breadcrumb component.
3. Ensure deep-linkable detail pages.

### Step 5 — Styling & UX

1. Apply Tailwind + MUI theme from style guide.
2. Configure Mantine React Table presets (sorting, filtering, density toggle).
3. Implement light/dark mode toggle.
4. Add responsive layouts for mobile/desktop.

### Step 6 — State Management

1. Use local state for UI controls.
2. Use server state (React Query/RTK Query) for data.
3. Store user preferences (table density, theme) in local storage.

### Step 7 — Quality & Tooling

1. Add unit tests for data-access libs.
2. Add Cypress or Playwright e2e tests for key flows.
3. Run Lighthouse and axe accessibility audits.

### Step 8 — Deployment

1. Add Dockerfile + docker-compose for API + DB + app.
2. Configure Nx build + serve targets for prod.
3. Deploy to chosen environment.

---

## 5. Suggested Enhancements

* Search bar for global search across entities.
* Tagging system for notes and todos.
* Export meetings/notes/todos to PDF.
* Calendar sync (Google, Outlook).
* WebSocket updates for real-time collaboration.

---

## 6. Implementation Checklist

* [ ] API CRUD for all entities.
* [ ] Data-access libs with hooks.
* [ ] Layout shell.
* [ ] Contacts feature.
* [ ] Meetings feature.
* [ ] Notes feature.
* [ ] Todos feature.
* [ ] Routing + breadcrumbs.
* [ ] Theming + responsive design.
* [ ] Testing + accessibility.
* [ ] Dockerized deployment.
