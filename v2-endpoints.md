# JetScale REST API v2

> Human-readable catalog of HTTP routes exposed under **`/api/v2`**, plus OAuth on **`/oauth/*`**. For request and response schemas, use the interactive OpenAPI UI on a running server.

This page groups every route by **module** (router prefix). Most handlers return the V2 JSON envelope **`BaseApiResponse`** (`meta` + `data`). Exceptions are called out in the tables below.

---

## Overview and conventions :id=overview-and-conventions

| Topic | Detail |
|--------|--------|
| **Base path** | `/api/v2` for modular monolith routers |
| **OAuth** | `/oauth/*` is mounted on the **root** app (not under `/api/v2`) |
| **Interactive spec** | OpenAPI at `/docs` and `/redoc` when the server runs |
| **MCP** | Model Context Protocol is mounted at `/mcp` (Streamable HTTP); see your internal `docs/mcp/usage.md` if applicable |
| **Route count** | 130 (GET / POST / PATCH / PUT; excludes implicit HEAD) |

**Authentication (typical):** Session and Bearer tokens issued by `/api/v2/auth` flows. Individual operations require the permissions configured for your tenant and role.

---

## Module index :id=module-index

| Module | Base path | On this page |
|--------|-----------|----------------|
| Authentication and identity | `/api/v2/auth` | [Go](#authentication-and-identity) |
| Cloud accounts and providers | `/api/v2/cloud` | [Go](#cloud-accounts-and-providers) |
| Integrations | `/api/v2/integrations` | [Go](#integrations) |
| Organization | `/api/v2/organization` | [Go](#organization) |
| Recommendations v2 | `/api/v2/recommendations-v2` | [Go](#recommendations-v2) |
| Remediation | `/api/v2/remediation` | [Go](#remediation) |
| Settings admin | `/api/v2/settings` | [Go](#settings-admin) |
| System | `/api/v2/system` | [Go](#system) |
| Temporary dashboard | `/api/v2/temp-dashboard` | [Go](#temporary-dashboard) |
| OAuth 2.1 | `/oauth` | [Go](#oauth-21) |

---

## Authentication and identity :id=authentication-and-identity

**Base path:** `/api/v2/auth`

### Invitations and password flows

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v2/auth/action/send-owner-signup-invitation` | Queue email inviting an owner to join an existing root company (superuser / invite permission). |
| `POST` | `/api/v2/auth/action/send-reset-password-invitation` | Queue password-reset invitation email for an existing user. |
| `POST` | `/api/v2/auth/action/send-signup-invitation` | Queue company-domain signup invitation email for a standard user. |
| `POST` | `/api/v2/auth/reset-password` | Set a user password by email (admin-style reset; requires appropriate user-management permission). |
| `POST` | `/api/v2/auth/reset-password-from-invitation` | Public flow: set password using the single-use token from a reset-password email. |

### Sign-in, session, and tokens

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v2/auth/sign-in` | Authenticate with email or username and password; issues tokens and session cookies. |
| `POST` | `/api/v2/auth/sign-out` | Invalidate the current session and clear auth cookies. |
| `POST` | `/api/v2/auth/sign-up` | Complete invitation-based registration; creates user, session, tokens, and cookies. |
| `POST` | `/api/v2/auth/token/refresh-token` | Exchange a refresh token for new access and refresh tokens. |
| `GET` | `/api/v2/auth/me` | Return the current user profile and session context (requires Bearer session). |

### Users

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v2/auth/users` | Paginated list of users with filters (tenant user directory). |
| `GET` | `/api/v2/auth/users/{user_id}` | Fetch one user by id. |
| `PATCH` | `/api/v2/auth/users/{user_id}` | Partially update user profile fields. |
| `POST` | `/api/v2/auth/users/{user_id}/action/delete` | Request deletion flow for a user. |
| `POST` | `/api/v2/auth/users/{user_id}/action/set-status` | Activate or deactivate a user account. |

### Roles and role assignments

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v2/auth/roles` | List roles for RBAC configuration. |
| `POST` | `/api/v2/auth/roles` | Create a new role. |
| `GET` | `/api/v2/auth/roles/{role_id}` | Fetch one role by id. |
| `PATCH` | `/api/v2/auth/roles/{role_id}` | Update role fields (name, permissions, and so on). |
| `POST` | `/api/v2/auth/roles/{role_id}/action/delete` | Soft-delete or deactivate a role. |
| `GET` | `/api/v2/auth/role-assignments` | List role assignments (user ↔ role, optional business unit) with filters and pagination. |
| `POST` | `/api/v2/auth/role-assignments` | Create a new role assignment. |
| `GET` | `/api/v2/auth/role-assignments/{assignment_id}` | Fetch one role assignment by id. |
| `PATCH` | `/api/v2/auth/role-assignments/{assignment_id}` | Partially update a role assignment. |

### Invitations and sessions (directory)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v2/auth/user-invitations` | List pending or historical user invitations. |
| `GET` | `/api/v2/auth/user-invitations/{user_invitation_id}` | Fetch one invitation by id. |
| `GET` | `/api/v2/auth/user-sessions` | List user sessions (audit / support). |
| `GET` | `/api/v2/auth/user-sessions/{user_session_id}` | Fetch one session record by id. |

### User activity (audit)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v2/auth/user-activity` | List security and audit activity events (filters via standard query params). |
| `GET` | `/api/v2/auth/user-activity/user/{user_id}` | List activity events for a specific user. |
| `GET` | `/api/v2/auth/user-activity/{user_activity_id}` | Fetch a single activity event. |

---

## Cloud accounts and providers :id=cloud-accounts-and-providers

**Base path:** `/api/v2/cloud`

### AWS discovery and sourced recommendations

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v2/cloud/aws/discovery/resources/` | List discovered AWS resources for connected accounts. |
| `POST` | `/api/v2/cloud/aws/discovery/resources/action/discover` | Start or record an AWS resource discovery job. |
| `GET` | `/api/v2/cloud/aws/sourced-recommendations` | List AWS Cost Optimization Hub (or sourced) recommendations ingested for the tenant. |
| `GET` | `/api/v2/cloud/aws/sourced-recommendations/{aws_sourced_recommendation_id}` | Fetch one sourced recommendation by id. |

### Azure credentials and discovery

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v2/cloud/azure/credentials/action/validate` | Validate Azure credential configuration before persisting. |
| `GET` | `/api/v2/cloud/azure/credentials/auth-methods` | List supported Azure authentication methods for linking. |
| `GET` | `/api/v2/cloud/azure/discovery/resources/` | List discovered Azure resources. |
| `POST` | `/api/v2/cloud/azure/discovery/resources/action/discover` | Trigger Azure resource discovery. |

### Integrations and accounts

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v2/cloud/cloud-account-integrations` | List cloud account integrations (credential bundles) with filters. |
| `POST` | `/api/v2/cloud/cloud-account-integrations` | Create a new integration record for a provider. |
| `GET` | `/api/v2/cloud/cloud-account-integrations/{cloud_account_integration_id}` | Fetch one integration by id. |
| `PATCH` | `/api/v2/cloud/cloud-account-integrations/{cloud_account_integration_id}` | Update integration metadata or credential fields. |
| `GET` | `/api/v2/cloud/cloud-accounts` | List linked cloud accounts with standard query params (`api_fields`, filters, and so on). |
| `POST` | `/api/v2/cloud/cloud-accounts` | Create a cloud account record (before or alongside linking). |
| `GET` | `/api/v2/cloud/cloud-accounts/{cloud_account_id}` | Fetch one cloud account by id (optional includes). |
| `PATCH` | `/api/v2/cloud/cloud-accounts/{cloud_account_id}` | Partially update cloud account fields. |
| `POST` | `/api/v2/cloud/link-cloud-account` | Attach a cloud account to an integration with provider-specific credentials. |

### In-product setup guides

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v2/cloud/cloud-accounts/documentation/aws/setup-guide` | Return AWS account linking documentation payload for the console. |
| `GET` | `/api/v2/cloud/cloud-accounts/documentation/azure/setup-guide` | Return Azure linking documentation payload for the console. |

### Custom recommendations and agent jobs

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v2/cloud/cloud-accounts/{cloud_account_id}/action/generate-custom-recommendation` | Trigger JetScale recommendation generation for resources under this account. |
| `GET` | `/api/v2/cloud/cloud-accounts/{cloud_account_id}/custom-recommendations` | List custom (JetScale-generated) recommendations scoped to the account. |
| `GET` | `/api/v2/cloud/recommendation-agent-operations` | List async recommendation-agent job operations (status and history). |
| `GET` | `/api/v2/cloud/recommendation-agent-operations/{recommendation_agent_operation_id}` | Fetch one recommendation-agent operation by id. |

---

## Integrations :id=integrations

**Base path:** `/api/v2/integrations`

### Bitbucket

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v2/integrations/bitbucket/me` | Connected Bitbucket user metadata. |
| `POST` | `/api/v2/integrations/bitbucket/me/action/connect` | Connect Bitbucket for repository and PR workflows. |
| `POST` | `/api/v2/integrations/bitbucket/pullrequests/action/create` | Open a pull request via the Bitbucket API. |
| `GET` | `/api/v2/integrations/bitbucket/repositories` | List repositories accessible to the integration. |
| `POST` | `/api/v2/integrations/bitbucket/repositories/action/create` | Create a Bitbucket repository. |
| `GET` | `/api/v2/integrations/bitbucket/repositories/{repository_slug}/branches` | List branches for PR and remediation flows. |
| `GET` | `/api/v2/integrations/bitbucket/workspaces` | List Bitbucket workspaces. |

### Jira

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v2/integrations/jira/issues/action/create` | Create a Jira issue from JetScale context. |
| `GET` | `/api/v2/integrations/jira/issues/priorities` | Priorities for issue creation UI. |
| `GET` | `/api/v2/integrations/jira/issues/types` | Issue types for issue creation UI. |
| `GET` | `/api/v2/integrations/jira/me` | Connected Jira user and profile metadata. |
| `POST` | `/api/v2/integrations/jira/me/action/connect` | Connect the current user's Atlassian account for Jira API access. |
| `GET` | `/api/v2/integrations/jira/projects` | List Jira projects visible to the connected account. |

---

## Organization :id=organization

**Base path:** `/api/v2/organization`

### Company bootstrap (superuser)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v2/organization/action/setup-company` | Create a non-root company with license (superuser only). |
| `POST` | `/api/v2/organization/action/setup-root-company` | Bootstrap the root company and license (superuser onboarding). |

### Companies

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v2/organization/companies` | List companies visible to the caller. |
| `POST` | `/api/v2/organization/companies` | Create a company record (standard org flow). |
| `GET` | `/api/v2/organization/companies/{company_id}` | Fetch one company by id. |
| `PATCH` | `/api/v2/organization/companies/{company_id}` | Update company fields. |

### Business units

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v2/organization/business-units` | List business units under the tenant. |
| `POST` | `/api/v2/organization/business-units` | Create a business unit. |
| `GET` | `/api/v2/organization/business-units/{business_unit_id}` | Fetch one business unit. |
| `PATCH` | `/api/v2/organization/business-units/{business_unit_id}` | Update business unit metadata. |
| `GET` | `/api/v2/organization/business-units/{business_unit_id}/users` | List users belonging to a business unit. |

---

## Recommendations v2 :id=recommendations-v2

**Base path:** `/api/v2/recommendations-v2`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v2/recommendations-v2/generate` | Run the V2 recommendation agent for up to N `resource_arns` from `ResourceDiscovery`. **Response:** legacy shape; **not** wrapped in `BaseApiResponse`. |

---

## Remediation :id=remediation

**Base path:** `/api/v2/remediation`

### Artifacts

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v2/remediation/artifacts` | List remediation artifacts. |
| `POST` | `/api/v2/remediation/artifacts` | Create a remediation artifact record. |
| `GET` | `/api/v2/remediation/artifacts/{artifact_id}` | Fetch one remediation artifact. |

### Session activities

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v2/remediation/session-activities` | List session activities. |
| `POST` | `/api/v2/remediation/session-activities` | Append a session activity row. |
| `GET` | `/api/v2/remediation/session-activities/{activity_id}` | Fetch one session activity. |

### Remediation sessions

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v2/remediation/sessions` | List remediation sessions (pagination, filters, field selection). |
| `POST` | `/api/v2/remediation/sessions` | Create a remediation session with explicit participants and optional custom recommendation link. |
| `POST` | `/api/v2/remediation/sessions/action/start` | Start a remediation session tied to an optional recommendation. **Response:** JSON `BaseApiResponse`. |
| `POST` | `/api/v2/remediation/sessions/action/stream-start` | Same intent as start; streams progress via **Server-Sent Events** (`text/event-stream`), **not** `BaseApiResponse`. |
| `GET` | `/api/v2/remediation/sessions/{session_id}` | Fetch one remediation session by UUID. |
| `PATCH` | `/api/v2/remediation/sessions/{session_id}` | Update session fields (status, participants, `ended_on`, and so on). |

### Session actions (chat, artifacts, status)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v2/remediation/sessions/{session_id}/action/add-activity` | Record a structured activity event on the session. |
| `POST` | `/api/v2/remediation/sessions/{session_id}/action/add-artifact` | Attach an artifact (file or URL) to the session. |
| `POST` | `/api/v2/remediation/sessions/{session_id}/action/remove-artifact` | Detach an artifact from the session. |
| `POST` | `/api/v2/remediation/sessions/{session_id}/action/send-message` | Post a non-streaming chat message to the remediation thread. |
| `POST` | `/api/v2/remediation/sessions/{session_id}/action/set-status` | Transition session lifecycle state (for example OPEN / CLOSED). |
| `POST` | `/api/v2/remediation/sessions/{session_id}/action/stream-send-message` | Send a chat message with **SSE** streaming replies (not the JSON envelope). |
| `POST` | `/api/v2/remediation/sessions/{session_id}/action/update-artifact` | Update artifact content or metadata. |

### Session exports

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v2/remediation/sessions/{session_id}/chat-history` | Return persisted chat and stream messages for the session. |
| `GET` | `/api/v2/remediation/sessions/{session_id}/clickops-md` | Download generated click-ops markdown for manual AWS changes. |

---

## Settings admin :id=settings-admin

**Base path:** `/api/v2/settings`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v2/settings/system` | Full system settings snapshot with value source resolution (**superuser**). |
| `POST`, `PUT` | `/api/v2/settings/system` | Merge-update system settings in the database with an audit trail (**superuser**). |
| `GET` | `/api/v2/settings/system/audit` | Paginated audit log of system setting changes (**superuser**). |
| `GET` | `/api/v2/settings/system/check` | Return whether the caller is a superuser (UI gating). |
| `POST` | `/api/v2/settings/system/reset` | Reset database-stored system settings to defaults (**superuser**). |
| `GET` | `/api/v2/settings/user` | Placeholder user preferences response (authenticated). |

---

## System :id=system

**Base path:** `/api/v2/system`

### Health and diagnostics

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v2/system/diagnostics` | Extended diagnostic payload for operators. |
| `GET` | `/api/v2/system/info` | Structured build and runtime info for diagnostics. |
| `GET` | `/api/v2/system/live` | Liveness probe: process is up. |
| `GET` | `/api/v2/system/ready` | Readiness probe: dependencies available for traffic. |
| `GET` | `/api/v2/system/version` | Application version string for clients and support. |

### System settings CRUD

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v2/system/settings` | List system settings with pagination and filters per the V2 query contract. |
| `POST` | `/api/v2/system/settings` | Create a new system setting record. |
| `POST` | `/api/v2/system/settings/action/seed-defaults` | Populate default rows in the system settings table (admin). |
| `GET` | `/api/v2/system/settings/by-key/{setting_key}` | Fetch a single setting by stable key. |
| `GET` | `/api/v2/system/settings/{setting_id}` | Fetch one system setting by id. |
| `PATCH` | `/api/v2/system/settings/{setting_id}` | Partial update of a system setting. |

### WebSocket subsystem checks

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v2/system/ws/info` | Same intent as `/api/v2/system/info` under the `/system/ws` prefix for WebSocket gateway checks. |
| `GET` | `/api/v2/system/ws/live` | Liveness-style check under the WebSocket subsystem path. |
| `GET` | `/api/v2/system/ws/ready` | Readiness-style check under the WebSocket subsystem path. |

---

## Temporary dashboard :id=temporary-dashboard

**Base path:** `/api/v2/temp-dashboard`

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v2/temp-dashboard/cost-usage` | Dashboard: aggregated cost and usage by category. |
| `GET` | `/api/v2/temp-dashboard/governance-gaps` | Dashboard: governance and compliance gap metrics by category. |
| `GET` | `/api/v2/temp-dashboard/open-recommendation` | Dashboard: counts of open recommendations by category. |
| `GET` | `/api/v2/temp-dashboard/resources-count` | Dashboard: discovered resource counts by category. |
| `GET` | `/api/v2/temp-dashboard/resources/health/details` | Dashboard: detailed resource health breakdown. |
| `GET` | `/api/v2/temp-dashboard/resources/health/summary` | Dashboard: resource health summary by category. |
| `GET` | `/api/v2/temp-dashboard/savings` | Dashboard: estimated savings from custom recommendations by category. |

---

## OAuth 2.1 :id=oauth-21

**Base path:** `/oauth` (root app, **not** under `/api/v2`)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/oauth/authorize` | OAuth 2.1 authorization UI: renders login form for authorization code with PKCE flow. |
| `POST` | `/oauth/authorize` | Submit OAuth login; on success redirects to `redirect_uri` with `code`. |
| `POST` | `/oauth/token` | Token endpoint: exchange `code` or `refresh_token` for JWT access tokens (MCP and third-party clients). **Response:** standard OAuth token JSON, **not** `BaseApiResponse`. |

---

## Related documentation

- [API Reference](api-reference.md) — product-facing overview and future public API notes.
- Open **`/docs`** or **`/redoc`** on your JetScale deployment for the authoritative OpenAPI document.
