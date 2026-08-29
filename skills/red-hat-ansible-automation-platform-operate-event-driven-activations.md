---
name: operate-event-driven-ansible-activations
description: >-
  Inspect, enable, disable and restart Event-Driven Ansible rulebook activations, and audit what
  rules fired and what actions they took.
api: Red Hat Event-Driven Ansible Controller API
generated: '2026-08-29'
method: generated
source: openapi/red-hat-ansible-automation-platform-event-driven-ansible-openapi.json
operations:
  - activations_list
  - activations_retrieve
  - activations_create
  - activations_enable_create
  - activations_disable_create
  - activations_restart_create
  - activation_instances_list
  - activation_instances_logs_list
  - audit_rules_list
  - audit_rules_actions_list
  - audit_rules_events_list
  - event_streams_list
  - decision_environments_list
  - rulebooks_list
---

# Operate Event-Driven Ansible activations

Base path: `https://<aap-gateway-host>/api/eda/v1/`. Authenticate with the Gateway-minted JWT in
`X-DAB-JW-TOKEN`, or a session cookie for browser callers.

## What an activation is

An `Activation` binds a rulebook to an event source and runs continuously inside a
`DecisionEnvironment` container image, firing actions when rules match. It is a long-running
consumer, not a request/response endpoint.

## Inspect before you touch

- `GET /activations/` (`activations_list`) — status and name of every activation.
- `GET /activation-instances/` (`activation_instances_list`) — individual executions.
- `GET /activation-instances/{id}/logs/` (`activation_instances_logs_list`) — what it did.
- `GET /audit-rules/` (`audit_rules_list`), then `/audit-rules/{id}/actions/` and
  `/audit-rules/{id}/events/` — the record of which rule fired on which event and what action ran.

## Control

- `POST /activations/{id}/disable/` (`activations_disable_create`) — stop consuming events. This
  is the reversal operation for the whole EDA surface.
- `POST /activations/{id}/enable/` (`activations_enable_create`) — resume.
- `POST /activations/{id}/restart/` (`activations_restart_create`) — bounce it.

**Disabling stops future events from firing actions. Events that already fired have already run
their playbooks, and those effects are not reversed.**

## Errors worth handling specifically

EDA has the richest error vocabulary in the platform, and 409 is doing real work:

- `409 Activation blocked while Workers offline.` — capacity, not a client bug. Retry later.
- `409 Project import or sync is already running.` — concurrency. Wait, do not retry immediately.
- `409 Decision Environment in use by Activations.` / `CredentialType in use by Credentials.` —
  referential integrity. Resolve the dependency first.
- `503 Project workers unavailable.` — infrastructure. Back off.
- `400 Rulebook not parseable.` — genuine client error; fix the rulebook.

See `errors/`.
