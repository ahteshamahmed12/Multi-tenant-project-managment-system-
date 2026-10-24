---
id: 1
title: Constitution TenantFlow
stage: constitution
date: 2026-10-24
surface: agent
model: claude-sonnet-4-6
feature: none
branch: main
user: ahtesham ahmed
command: sp.constitution
labels: ["constitution", "multi-tenant", "saas", "governance"]
links:
  spec: null
  ticket: null
  adr: null
  pr: null
files:
  - .specify/memory/constitution.md
  - history/prompts/constitution/1-constitution-tenantflow.constitution.prompt.md
tests:
  - null
---

## Prompt

Create the constitution for a multi-tenant project management SaaS (boards,
sprints, tasks, comments, roles, notifications, reporting).
Stack: FastAPI, PostgreSQL, SQLAlchemy (async), Alembic, Pydantic, Redis,
JWT auth, React + TypeScript + Tailwind frontend.

## I. Tenant Isolation (NON-NEGOTIABLE)
- Tenancy model: shared database, shared schema, one `tenant_id` per row.
- Every tenant-owned table MUST have `tenant_id` NOT NULL, indexed, and
  included in composite unique constraints.
- `tenant_id` comes ONLY from the verified JWT/server context, never from
  request bodies, query params, or URLs.
- All database access goes through one tenant-scoped repository layer.
  No raw queries that skip the tenant filter.
- PostgreSQL Row-Level Security is enabled as a second line of defense.
- Redis keys, cache entries, WebSocket channels, file paths, and background
  jobs MUST be namespaced by tenant.
- A resource in another tenant returns 404, never 403.
- Every endpoint MUST have a cross-tenant access test that proves isolation.

## II. Security & Access Control
- JWT access + refresh tokens; claims carry user_id, tenant_id, role.
- RBAC (owner, admin, project manager, member, viewer) enforced server-side
  on every endpoint. Never trust the frontend for permissions.
- Passwords hashed with bcrypt/argon2. No secrets in code; env vars only.
- Rate limiting per tenant and per user.
- All create/update/delete actions write to an audit log (who, what, when,
  tenant).
- Auth, tenancy, and permission code MUST be reviewed by a human before
  merge.

## III. Layered Architecture
- Router -> Service -> Repository. No business logic in routers, no HTTP
  concerns in services.
- Pydantic schemas at every API boundary; never expose ORM models directly.
- Database changes only via Alembic migrations. Never edit schema by hand.
- Async/await end to end.

## IV. Test-First Quality
- Write tests before implementation for auth, tenancy, and RBAC.
- pytest, with unit tests for services and integration tests for endpoints.
- A feature is not done until its tests pass, including permission-denied
  and cross-tenant cases.

## V. API Standards
- Versioned under /api/v1. OpenAPI docs always accurate.
- Consistent error format. Pagination is mandatory on every list endpoint.
- Validate all input; fail with clear, safe error messages.

## VI. Performance & Data Integrity
- Indexes lead with tenant_id, for example (tenant_id, project_id, status).
- No N+1 queries. Use eager loading or explicit joins.
- Use transactions for multi-step writes. Soft-delete where history matters.

## VII. Observability
- Structured JSON logs with request_id, tenant_id, user_id.
- Health check endpoint. Errors are never silently swallowed.

## VIII. Workflow Rules
- No code without an approved spec, plan, and tasks.
- Small, reviewable tasks. One concern per change.
- Definition of Done: tests pass, migration included, docs updated,
  no cross-tenant leaks, reviewed.

## Response snapshot

Created TenantFlow Constitution v1.0.0 at `.specify/memory/constitution.md` by mapping the 8 user-provided sections onto the 5-principle + 2-section + governance template structure. Mapped sections I–V as Core Principles 1–5, moved VI–VII into the two template section slots, and VIII into the workflow/governance context. Added a Sync Impact Report HTML comment at the top documenting version, added/removed sections, and template validation status. All placeholders resolved; no TODOs remain.

## Outcome

- ✅ Impact: Established project-wide governance constitution for multi-tenant SaaS
- 🧪 Tests: N/A (governance artifact)
- 📁 Files: `.specify/memory/constitution.md` (created), `history/prompts/constitution/1-constitution-tenantflow.constitution.prompt.md` (this PHR)
- 🔁 Next prompts: `/sp.spec` to create feature specs; `/sp.plan` and `/sp.tasks` to implement features against this constitution
- 🧠 Reflection: Template had 5 principle slots + 2 section slots + governance. Used all 5 principle slots for sections I–V, both section slots for VI + VII, folded VIII into Governance. Version is 1.0.0 as initial ratification.

## Evaluation notes (flywheel)

- Failure modes observed: None
- Graders run and results (PASS/FAIL): N/A
- Prompt variant (if applicable): None
- Next experiment (smallest change to try): N/A
