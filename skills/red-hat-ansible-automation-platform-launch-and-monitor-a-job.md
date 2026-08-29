---
name: launch-and-monitor-an-ansible-job
description: >-
  Launch a Red Hat Ansible Automation Platform job template, watch it to completion, read its
  output, and cancel or relaunch it. Use this whenever an agent needs to make AAP actually run
  automation rather than just read configuration.
api: Red Hat Ansible Automation Controller API
generated: '2026-08-29'
method: generated
source: openapi/red-hat-ansible-automation-platform-automation-controller-openapi.json
operations:
  - job_templates_list
  - job_templates_retrieve
  - job_templates_launch_retrieve
  - job_templates_launch_create
  - jobs_retrieve
  - jobs_stdout_retrieve
  - jobs_job_events_list
  - jobs_cancel_retrieve
  - jobs_cancel_create
  - jobs_relaunch_create
---

# Launch and monitor an Ansible job

Base path on an AAP 2.5+ installation: `https://<aap-gateway-host>/api/controller/v2/`.
Authenticate with `Authorization: Bearer <aap_token>` (see `authentication/`).

## 1. Find the template

`GET /job_templates/` (`job_templates_list`). Filter with `?search=<text>` and page with
`page` / `page_size`. Read `id` and `name` from `results[]`.

## 2. Pre-flight BEFORE you launch — this is not optional

`GET /job_templates/{id}/launch/` (`job_templates_launch_retrieve`) returns what a launch would
require: prompt-on-launch fields, `survey_enabled` and the survey spec, credentials that need
passwords, and whether the template can be launched at all by this principal.

This is the only dry run AAP gives you for the highest-consequence write in the API. Read it
first. If you need a rehearsal of the playbook itself rather than of the launch, set
`job_type: "check"` — Ansible check mode — on the template or in the launch payload.

## 3. Launch

`POST /job_templates/{id}/launch/` (`job_templates_launch_create`). The response carries the new
job `id`.

**There is no idempotency.** AAP declares no `Idempotency-Key` anywhere in its contract. If you
retry this POST because a response timed out, you will start a second job. Before retrying,
`GET /jobs/?job_template=<id>` and check whether your launch already landed.

## 4. Watch it

- `GET /jobs/{id}/` (`jobs_retrieve`) — `status` moves through pending, waiting, running, then
  successful / failed / error / canceled.
- `GET /jobs/{id}/job_events/` (`jobs_job_events_list`) — structured per-task events.
- `GET /jobs/{id}/stdout/` (`jobs_stdout_retrieve`) — raw output; this returns text, not JSON.

Poll. There is no server-sent-event or websocket surface in the contract.

## 5. Cancel — and know what cancelling does not do

`GET /jobs/{id}/cancel/` (`jobs_cancel_retrieve`) returns `{"can_cancel": true|false}`. Check it
first; the window is state-based, not time-based. Then `POST /jobs/{id}/cancel/`
(`jobs_cancel_create`).

Cancelling stops further execution. **It does not undo what the playbook already did to the
managed hosts.** AAP has no rollback. If reversal matters, it has to be a second playbook.

## 6. Relaunch is not undo

`POST /jobs/{id}/relaunch/` (`jobs_relaunch_create`) re-runs the same job with the same inputs.
It is a forward operation. Never reach for it expecting a rollback.

## Error handling

The Controller spec declares no 4xx/5xx responses, so treat errors as the Django REST Framework
shape: `{"detail": "..."}` for non-field errors, or an object keyed by field name for validation
errors. See `errors/`.
