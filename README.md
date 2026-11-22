# Departmental App Brief & OpenSpec Starter

A structured workflow and specification template for taking **departmental management applications** (< 40 users — CRUD, queries, reporting, and CSV/Excel import/export) from an initial business interview to a spec-driven implementation using **Antigravity** and **OpenSpec**.

## Quick Start (Using as a Template Repository)

Keep this repository clean as your **agent & architecture template**, and create a separate copy for each new application:

1. **Clone or create a repo from this template** into a new project folder (e.g., `facturas-app/`).
2. **Open the new project folder** in Antigravity.
3. **Run `/dept-app-brief`** to conduct the functional interview and generate `app-brief.md` in that project's root.
4. **Run `/opsx-explore`** to resolve open technical questions using `app-brief.md` and `openspec/specs/architecture/spec.md`.
5. **Run `/opsx-propose` and `/opsx-apply`** to generate OpenSpec change artifacts and build the application inside that isolated repository.

---

## Repository Structure

```text
.
├── .agents/
│   ├── rules/
│   │   └── app-brief-openspec-bridge.md   # Auto-connects app-brief.md with openspec-explore
│   ├── skills/
│   │   └── openspec-*/                    # OpenSpec skills (explore, propose, apply, update, sync, archive)
│   └── workflows/
│       ├── dept-app-brief.md              # Functional analyst interview workflow
│       └── opsx-*.md                      # OpenSpec slash-command workflows (/opsx-*)
├── openspec/
│   ├── config.yaml                        # Mandatory tech stack, architecture, and OpenSpec rules
│   ├── specs/
│   │   └── architecture/
│   │       └── spec.md                    # Core Departmental Architecture specification
│   └── changes/                           # OpenSpec change proposals, designs, and tasks
└── app-brief-example.md                   # Example functional brief output
```

---

## 1. The `dept-app-brief.md` Workflow

Defined in [`.agents/workflows/dept-app-brief.md`](.agents/workflows/dept-app-brief.md) and invoked via `/dept-app-brief`, this workflow turns the agent into a **Functional Analyst** that gathers requirements in business terms (avoiding unnecessary technical jargon) and produces a single `app-brief.md` document.

### How It Works
1. **Pre-Interview Repository Inspection:** Before asking any questions, the agent scans the repository for existing SQL schemas (`*.sql`) or sample CSV files (`*.csv`). If found, it extracts entities, fields, constraints, relationships, and CSV columns/delimiters to use as a baseline proposal rather than asking from scratch.
2. **Incremental Interview (5 Phases, 2–3 questions per turn):**
   - **Phase 1 — Context & Actors:** Business problem, purpose, user roles, and general permissions.
   - **Phase 2 — Domains & Business Rules:** Main entities, relationships (`1:N`, `N:M`), key fields/validations, state lifecycles, and logical vs. physical deletion.
   - **Phase 3 — Screens & Operations:** CRUD forms, query tables with filters, dashboards, and bulk/state-transition actions.
   - **Phase 4 — Reports & Import/Export:** KPIs/reports (PDF/Excel) and CSV import rules (required columns, delimiter, duplicate/error handling).
   - **Phase 5 — Out of Scope (v1) & Confirmation:** Explicit v1 boundaries and final summary before generating the document.
3. **Deliverable (`app-brief.md`):** Generates a 6-section functional brief covering Objective & Scope, Roles & Permissions, Domains & Data Model, Forms & Screens, Reports & Data Import/Export, and **Section 6: Open Questions & Points to Explore (`openspec-explore`)**.

See [`app-brief-example.md`](app-brief-example.md) for a complete sample output.

---

## 2. How It Interacts with OpenSpec

The handoff between functional discovery and technical design is automated by the workspace rule [`.agents/rules/app-brief-openspec-bridge.md`](.agents/rules/app-brief-openspec-bridge.md) (`trigger: model_decision`):

```text
[/dept-app-brief] ──▶ app-brief.md ──▶ (bridge rule) ──▶ [/opsx-explore] ──▶ [/opsx-propose] ──▶ [/opsx-apply]
```

1. **Automatic Brief Detection:** When you start `/opsx-explore` (or `openspec-explore`), the bridge rule instructs the agent to locate and read `app-brief.md` before asking discovery questions. If no brief exists yet, it suggests running `/dept-app-brief` first.
2. **No Repeated Interview:** Sections 1–5 of `app-brief.md` are treated as the validated functional baseline and mapped directly to technical concerns:
   - *Section 1 (Scope)* → Strict v1 boundaries.
   - *Section 2 (Roles)* → Authentication and RBAC strategy.
   - *Section 3 (Domains)* → Prisma data model, constraints, state transitions, and soft-delete policy.
   - *Section 4 (Screens)* → RSC vs. Client Components, filters, and bulk actions.
   - *Section 5 (Reports/CSV)* → PDF/Excel generation and CSV validation/import pipeline.
3. **Priority Exploration Agenda:** Section 6 (`Dudas Abiertas y Puntos a Explorar`) becomes the primary agenda for `/opsx-explore`.
4. **Transition to Implementation:** Once all open questions are resolved into explicit technical decisions, use `/opsx-propose` to generate OpenSpec change artifacts (`proposal`, `design`, `tasks`) and `/opsx-apply` to implement them.

---

## 3. Reference Architecture (`openspec/config.yaml` & `architecture/spec.md`)

All applications designed and built with this template follow a standardized **modular monolith** architecture defined in [`openspec/config.yaml`](openspec/config.yaml) and [`openspec/specs/architecture/spec.md`](openspec/specs/architecture/spec.md).

### Mandatory Tech Stack
| Layer | Technology |
|---|---|
| **Framework** | Next.js (App Router) with strict TypeScript (`strict: true`) |
| **Styling & UI** | Tailwind CSS + shadcn/ui (Radix UI) + Lucide icons |
| **Database & ORM** | PostgreSQL + Prisma (`prisma/schema.prisma`) |
| **Validation** | Zod (all forms, Server Actions, and endpoints) |
| **Authentication** | Auth.js (NextAuth v5) with corporate SSO only (Entra ID / OIDC) |
| **Deployment** | Single Docker container (`output: "standalone"` in `next.config.ts`) |

### Architectural Patterns & Simplicity Rules
- **Modular Monolith:** Single repository; no microservices, message queues (Redis/RabbitMQ), or complex global state (Redux) unless explicitly justified.
- **Data Access:**
  - **Reads:** React Server Components (RSC) querying Prisma directly from `src/lib/data/`.
  - **Mutations:** Next.js Server Actions in `src/actions/` (validated with Zod, session-checked, and using `revalidatePath`). Route Handlers (`/api/*`) are reserved for webhooks or external integrations.
- **Standard Folder Layout:** `src/app/` (routes/layouts), `src/components/ui/` (shadcn/ui), `src/components/` (business components), `src/actions/`, `src/lib/db.ts` (Prisma singleton), `src/lib/auth.ts` (Auth.js & authorization helpers), and `src/lib/validations/` (shared Zod schemas).

### Core Architectural Requirements (`architecture/spec.md`)
1. **End-to-End Type Safety:** Strict TypeScript across database, server, and client; all incoming payloads validated with Zod before touching Prisma.
2. **Delegated Authentication (SSO):** Corporate OIDC / Entra ID via Auth.js; unauthenticated requests are rejected or redirected, and local passwords are never stored.
3. **Modular Monolith Data Access:** All PostgreSQL access goes through Prisma Client on the server (RSC for reads, Server Actions for writes).
4. **Standardized Container Deployment:** Builds with Next.js `standalone` output into a minimal, self-contained Docker image with no runtime dev dependencies.

### OpenSpec Artifact Rules (`config.yaml`)
- **`proposal`:** Must fit within the Next.js + PostgreSQL monolith and include a *Database Impact* section if `prisma/schema.prisma` changes.
- **`design`:** Must specify exact Prisma models/indexes, Zod schemas, Server Actions, RSC vs. `'use client'` boundaries, and SSO role-based access per route/action.
- **`tasks`:** Ordered strictly as: `1) Prisma schema & migration` → `2) Zod validations` → `3) Server Actions & data logic` → `4) UI with shadcn/ui` → `5) Type checking (tsc --noEmit) & build`.
