<!-- SYNC IMPACT REPORT
Version change: — (initial) → 1.0.0
Modified principles (old title → new title):
  [PRINCIPLE_1_NAME] → I. Tenant Isolation (NON-NEGOTIABLE)
  [PRINCIPLE_2_NAME] → II. Security & Access Control
  [PRINCIPLE_3_NAME] → III. Layered Architecture
  [PRINCIPLE_4_NAME] → IV. Test-First Quality
  [PRINCIPLE_5_NAME] → V. API Standards
Added sections:
  Section 2: VI. Performance & Data Integrity + VII. Observability
  Section 3: VIII. Workflow Rules
Removed sections: none
Templates requiring updates:
  ✅ .specify/templates/plan-template.md — "Constitution Check" placeholder remains generic; no changes needed for v1.0.0
  ✅ .specify/templates/spec-template.md — no mandatory constitution sections referenced in template; no changes needed
  ✅ .specify/templates/tasks-template.md — task categorization is generic; no changes needed
  ✅ .specify/templates/adr-template.md — no constitution references in template; no changes needed
  ✅ .specify/templates/checklist-template.md — no constitution references in template; no changes needed
  ✅ .specify/templates/agent-file-template.md — no constitution references in template; no changes needed
  ✅ .specify/templates/phr-template.prompt.md — no constitution references in template; no changes needed
Follow-up TODOs: None — all placeholders resolved.
-->

# TenantFlow Constitution

## Core Principles

### I. Tenant Isolation (NON-NEGOTIABLE)
Every tenant-owned table MUST have `tenant_id` NOT NULL, indexed, and included in composite unique constraints. Tenancy model: shared database, shared schema, one `tenant_id` per row. `tenant_id` comes ONLY from the verified JWT/server context, never from request bodies, query params, or URLs. All database access goes through one tenant-scoped repository layer; no raw queries that skip the tenant filter. PostgreSQL Row-Level Security is enabled as a second line of defense. Redis keys, cache entries, WebSocket channels, file paths, and background jobs MUST be namespaced by tenant. A resource in another tenant returns 404, never 403. Every endpoint MUST have a cross-tenant access test that proves isolation.

### II. Security & Access Control
JWT access + refresh tokens; claims carry user_id, tenant_id, role. RBAC (owner, admin, project manager, member, viewer) MUST be enforced server-side on every endpoint; the frontend is never trusted for permissions. Passwords hashed with bcrypt/argon2. No secrets in code; env vars only. Rate limiting per tenant and per user. All create/update/delete actions write to an audit log (who, what, when, tenant). Auth, tenancy, and permission code MUST be reviewed by a human before merge.

### III. Layered Architecture
Router -> Service -> Repository with no business logic in routers and no HTTP concerns in services. Pydantic schemas at every API boundary; ORM models are never exposed directly. Database changes only via Alembic migrations; schema is never edited by hand. Async/await end to end.

### IV. Test-First Quality
Tests MUST be written before implementation for auth, tenancy, and RBAC. pytest with unit tests for services and integration tests for endpoints. A feature is not done until its tests pass, including permission-denied and cross-tenant cases.

### V. API Standards
Versioned under `/api/v1`; OpenAPI docs always accurate. Consistent error format. Pagination is mandatory on every list endpoint. All input validated; fail with clear, safe error messages.

## VI. Performance & Data Integrity

Indexes lead with `tenant_id` (e.g., `(tenant_id, project_id, status)`). No N+1 queries; use eager loading or explicit joins. Transactions for multi-step writes. Soft-delete where history matters.

## VII. Observability

Structured JSON logs with `request_id`, `tenant_id`, `user_id`. Health check endpoint. Errors are never silently swallowed.

## VIII. Workflow Rules

No code without an approved spec, plan, and tasks. Small, reviewable tasks; one concern per change. Definition of Done: tests pass, migration included, docs updated, no cross-tenant leaks, reviewed.

## Governance

Constitution supersedes all other practices. Amendments require documentation, approval, and a migration plan. All PRs/reviews MUST verify compliance with these principles. Complexity must be justified. Use `CLAUDE.md` for runtime development guidance. Version changes follow semantic versioning: MAJOR for backward-incompatible principle removals or redefinitions, MINOR for new or materially expanded sections, PATCH for non-semantic clarifications. Auth, tenancy, and permission code requires human review before merge.

**Version**: 1.0.0 | **Ratified**: 2026-10-24 | **Last Amended**: 2026-10-24
