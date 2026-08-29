---
name: audit-aap-platform-access
description: >-
  Read who can do what in Red Hat Ansible Automation Platform — users, teams, organizations, role
  assignments, identity providers and the activity stream — through the Platform Gateway.
api: Red Hat Ansible Automation Platform Gateway API
generated: '2026-08-29'
method: generated
source: openapi/red-hat-ansible-automation-platform-platform-gateway-openapi.json
operations:
  - users_list
  - users_retrieve
  - teams_list
  - teams_retrieve
  - organizations_list
  - role_definitions_list
  - role_user_assignments_list
  - role_team_assignments_list
  - authenticators_list
  - authenticator_maps_list
  - applications_list
  - tokens_list
  - activitystream_list
  - status_retrieve
  - me_list
---

# Audit platform access

Base path: `https://<aap-gateway-host>/api/gateway/v1/`. From AAP 2.5 the Gateway is the single
front door — identity lives here, not in the Controller.

## Establish who you are first

`GET /me/` (`me_list`) tells you which principal the token belongs to. Tool visibility in Red
Hat's own MCP server is filtered by exactly these permissions, so this is the honest starting
point for any access question.

## Read the access graph

- `GET /organizations/` — the tenancy boundary almost everything belongs to.
- `GET /teams/` and `GET /users/`.
- `GET /role_definitions/` — the named permission sets.
- `GET /role_user_assignments/` and `GET /role_team_assignments/` — who actually holds what.

**Use the assignment model, not the membership endpoints.** `organizations_users_list`,
`organizations_admins_associate_create`, `teams_users_disassociate_create` and 11 siblings are
marked `deprecated: true` in this spec — 14 operations in total. They are superseded by role
assignments.

## Identity federation

- `GET /authenticators/` (`authenticators_list`) — the configured LDAP / SAML / OIDC / local
  providers.
- `GET /authenticator_maps/` (`authenticator_maps_list`) — how provider claims map to AAP teams
  and organizations. This is where an access review usually finds surprises.

## OAuth surface

- `GET /applications/` (`applications_list`) and `GET /tokens/` (`tokens_list`) — issued OAuth
  applications and tokens. Note the Gateway still offers the resource-owner **password** grant
  alongside authorization_code; OAuth 2.1 removes it. Flag it in any review.
- Scopes are only `read` and `write` — there is no fine-grained scope surface. Authorization is
  done by AAP RBAC after the token is accepted, not by the scope on it. See `scopes/`.

## Evidence trail

`GET /activitystream/` (`activitystream_list`) on the Gateway, and
`GET /api/controller/v2/activity_stream/` on the Controller, record who changed what. These are
the closest thing AAP has to request tracing — no `X-Request-Id` or `traceparent` header appears
anywhere in the contract.
