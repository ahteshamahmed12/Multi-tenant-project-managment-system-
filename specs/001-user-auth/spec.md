# Feature Specification: Tenant Onboarding and Authentication

**Feature Branch**: `001-user-auth`  
**Created**: 2026-10-24  
**Status**: Draft  
**Input**: Tenant onboarding and authentication for a multi-tenant project management SaaS

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Workspace Creation and Ownership (Priority: P1)

A new user registers with email and password and creates a private workspace in a single flow, becoming its Owner.

**Why this priority**: This is the entry point for all new teams; without it no one can use the product.

**Independent Test**: Can be fully tested by completing registration with a unique email and verifying a new workspace exists with the user as its Owner.

**Acceptance Scenarios**:

1. **Given** a visitor on the registration page, **When** they submit a valid email, password meeting minimum strength, and a unique workspace name, **Then** a user account and workspace are created and the user is authenticated as Owner.
2. **Given** a visitor on the registration page, **When** they submit a workspace name already in use, **Then** the system rejects the name and prompts for a different name.
3. **Given** a visitor on the registration page, **When** they submit an email already registered in the system, **Then** the system indicates the email is already in use and suggests login.

---

### User Story 2 - Login and Session Management (Priority: P2)

A registered user logs in with email and password, receives access and refresh tokens, and can log out from the current device.

**Why this priority**: Returning users need secure, persistent access to their workspaces.

**Independent Test**: Can be fully tested by logging in with valid credentials and verifying token issuance, then logging out and confirming the access token is invalidated.

**Acceptance Scenarios**:

1. **Given** a user with a verified account, **When** they submit correct credentials, **Then** the system returns access and refresh tokens and grants access based on their role.
2. **Given** a user with a verified account, **When** they submit incorrect credentials 5 times within 15 minutes, **Then** the system locks login attempts for both the user account and the source IP address for 15 minutes.
3. **Given** a logged-in user, **When** they request logout, **Then** the access and refresh tokens for that session are invalidated.

---

### User Story 3 - Workspace Member Invitation (Priority: P3)

An Owner or Admin invites people by email and assigns them a role (admin, member, or viewer).

**Why this priority**: Teams need to collaborate; invitations are the bridge from solo to multi-user usage.

**Independent Test**: Can be fully tested by inviting an email address from a workspace and verifying that a time-limited, single-use invite link is emailed to that address.

**Acceptance Scenarios**:

1. **Given** an Owner or Admin in a workspace, **When** they invite an email not yet in the workspace, **Then** an invitation with a 7-day expiry is created and emailed.
2. **Given** an Owner or Admin invites an email belonging to a user already in the workspace, **When** the invitation is processed, **Then** the system indicates the user already has access and no new invite is created.
3. **Given** an Owner or Admin, **When** they attempt to invite themselves with a lower role, **Then** the system preserves their current Owner role.

---

### User Story 4 - Invite Acceptance and Join (Priority: P4)

An invited person accepts the invite, sets their password, and joins the workspace with the assigned role.

**Why this priority**: Without acceptance, invited collaborators cannot access the workspace.

**Independent Test**: Can be fully tested by following an invite link, setting a password, and verifying workspace membership with the correct role.

**Acceptance Scenarios**:

1. **Given** a user with a valid, unused, unexpired invite link, **When** they set a password meeting minimum strength, **Then** their account is created (or linked if the email already has an account) and they become a member of the workspace with the assigned role.
2. **Given** a user clicks an invite link that has expired, **When** they attempt to accept, **Then** the system indicates the invite has expired and offers to request a new invite.
3. **Given** a user clicks an invite link already used, **When** they attempt to accept, **Then** the system indicates the invite is no longer valid.

---

### User Story 5 - Member Role Management and Removal (Priority: P5)

An Owner or Admin changes a member's role or removes them from the workspace.

**Why this priority**: Teams evolve; administrators need tools to adjust permissions and revoke access.

**Independent Test**: Can be fully tested by changing a member's role and verifying permission changes, then removing a member and confirming immediate loss of access.

**Acceptance Scenarios**:

1. **Given** an Owner or Admin, **When** they change a member's role, **Then** the new role takes effect immediately and the change is recorded in the audit log.
2. **Given** an Owner, **When** they attempt to change the last remaining Owner to a lower role, **Then** the system prevents the action and indicates the workspace must retain at least one Owner.
3. **Given** an Owner or Admin removes a member, **When** the removal completes, **Then** the member loses access immediately and the removal is recorded in the audit log.

---

### User Story 6 - Password Reset (Priority: P6)

A user resets their password through an emailed link.

**Why this priority**: Users forget passwords; a secure self-service reset reduces support burden.

**Independent Test**: Can be fully tested by requesting a reset link, opening it, setting a new password, and logging in with the new credentials.

**Acceptance Scenarios**:

1. **Given** a user with a registered email, **When** they request a password reset, **Then** a time-limited reset link is emailed to that address.
2. **Given** a user clicks an expired reset link, **When** they attempt to reset, **Then** the system indicates the link has expired.
3. **Given** a user resets their password, **When** they attempt to log in with the new password, **Then** login succeeds.

---

### User Story 7 - Workspace Switching (Priority: P7)

A user who belongs to multiple workspaces switches between them to access each workspace's projects and tasks.

**Why this priority**: Team members often collaborate across multiple workspaces; seamless switching improves productivity.

**Independent Test**: Can be fully tested by being a member of two workspaces and switching between them, verifying workspace-scoped data isolation.

**Acceptance Scenarios**:

1. **Given** a user who is a member of multiple workspaces, **When** they switch to a different workspace, **Then** the application context changes to that workspace and only that workspace's data is visible.
2. **Given** a logged-in user, **When** they attempt to access a workspace by ID they do not belong to, **Then** the system returns 404 and no data is revealed.
3. **Given** a user who has switched workspaces, **When** they navigate back to a workspace they belong to, **Then** their previous context is restored.

---

### Edge Cases

- Registering with an email that already exists in the same workspace: system prevents duplicate membership.
- Accepting an expired or already-used invite: system rejects and prompts for a new invite.
- The last Owner tries to leave or demote themselves: system prevents, ensuring the workspace always has at least one Owner.
- A user is invited to a workspace they already belong to: system prevents duplicate invitation.
- A user guesses another workspace's ID: system returns 404 with no data leakage.
- Login with repeated failed attempts: rate limiting locks the account for a defined period.
- New user does not verify email within 24 hours: account remains unverified and login is blocked until verification completes.
- Invite acceptance when the user has an existing account: email is verified and role is assigned without creating a duplicate account.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST allow new users to register with email and password and send a verification email before login is enabled.
- **FR-002**: System MUST enforce minimum password strength on registration and password reset.
- **FR-003**: System MUST create a workspace with a globally unique URL-safe slug during registration, with the registering user as Owner. The slug serves as the tenant identifier for routing.
- **FR-004**: System MUST issue JWT access and refresh tokens upon successful login.
- **FR-005**: System MUST validate tokens on every protected request, extracting tenant_id and role from claims.
- **FR-006**: System MUST prevent a user from registering an email already in use.
- **FR-007**: System MUST allow Owners and Admins to invite users by email with an assigned role.
- **FR-008**: System MUST send time-limited (7-day expiry) invite links via email.
- **FR-009**: System MUST prevent inviting an email already belonging to a workspace member.
- **FR-010**: System MUST allow invitees to accept an invite, set a password, and join the workspace.
- **FR-011**: System MUST prevent acceptance of expired or already-used invites.
- **FR-012**: System MUST allow Owners and Admins to change a member's role.
- **FR-013**: System MUST allow Owners and Admins to remove members, with immediate revocation of access.
- **FR-014**: System MUST prevent demotion or removal of the last Owner in a workspace.
- **FR-015**: System MUST record role changes and member removals in an audit log.
- **FR-016**: System MUST allow users to request a password reset link via email.
- **FR-017**: System MUST rate-limit repeated failed login attempts per user.
- **FR-018**: System MUST allow users belonging to multiple workspaces to list their workspaces via an endpoint and switch to another workspace via token refresh, updating the active tenant_id in the JWT.
- **FR-019**: System MUST return 404 for any resource belonging to a workspace the user does not belong to.
- **FR-020**: System MUST revoke the refresh token on logout so it cannot be used to obtain new access tokens. Access tokens expire naturally within their 15-minute TTL.

### Key Entities

- **Workspace (Tenant)**: A private, isolated project management workspace identified by a unique URL-safe slug. Contains projects, sprints, tasks, and members. Every record is scoped to a workspace via `tenant_id`.
- **User**: An authenticated individual with an email and password. A user can belong to multiple workspaces with different roles.
- **WorkspaceMember**: The association between a User and a Workspace, carrying a role (owner, admin, member, viewer). The same user has a separate membership per workspace.
- **Invite**: A time-limited, single-use token sent to an email address granting membership to a specific workspace with an assigned role. Expires after 7 days.
- **AuditLog**: Immutable record of create/update/delete actions within a workspace, capturing who performed the action, what changed, and when.
- **Token (JWT)**: Access and refresh tokens carrying `user_id`, `tenant_id`, and `role` claims. Access tokens have a short TTL; refresh tokens have a longer TTL and can be revoked.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: New teams can complete workspace creation and invite their first teammate in under 3 minutes from start to email delivery.
- **SC-002**: No HTTP request can read or modify data from a workspace the authenticated user does not belong to (100% isolation guarantee).
- **SC-003**: Repeated failed login attempts (5+ within 15 minutes) result in a lockout period of at least 15 minutes.
- **SC-004**: All role changes and member removals produce an audit log entry within 2 seconds of the action.
- **SC-005**: Invite links expire within 7 days of creation; expired invites cannot grant workspace access.
- **SC-006**: Password reset links expire within 24 hours of generation.

### Assumptions

- Invitations are delivered via an external email service (provider to be determined during planning).
- The frontend is a single-page application that stores JWT tokens in memory or secure HTTP-only cookies.
- Workspace slugs are auto-generated from the workspace name, with numeric or suffix disambiguation if needed.
- Rate limiting is enforced via Redis for both per-user and per-IP dimensions.
- JWT access tokens have a 15-minute TTL; refresh tokens have a 30-day TTL and can be revoked per-session.

## Clarifications

### Session 2026-10-24

- Q: Should new users verify their email before they can log in, or should they be able to log in immediately after registration? → A: Email verification REQUIRED before login (Option A). New users receive a verification link on registration and cannot sign in until verified. Verification links expire within 24 hours.

- Q: When repeated failed login attempts occur, does the lockout apply to the user account, the IP address, or both? → A: Lockout applies to BOTH user account and IP address (Option A). 5 failed attempts in 15 min triggers a 15-minute lockout on both dimensions.

- Q: Is the workspace slug unique globally across the SaaS platform, or scoped to an organization? → A: Workspace slug is globally unique and serves as the tenant identifier (Option A). Routing resolves workspace slug → tenant_id.

- Q: On logout, do we blacklist the access token server-side, or only revoke the refresh token? → A: Only revoke the refresh token; access token expires naturally within its 15-minute TTL (Option A). Stateless access tokens.

- Q: When a user belongs to multiple workspaces, how do they discover and switch between them — single tenant_id JWT or multi-tenant claims? → A: JWT carries a single tenant_id (active workspace); a `/workspaces` endpoint lists memberships; switching triggers token refresh with the new tenant_id (Option A).
