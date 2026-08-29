---
name: publish-and-certify-ansible-collections
description: >-
  Search, publish, approve and move Ansible Content Collections through Red Hat Automation Hub,
  including the async task lifecycle every write returns.
api: Red Hat Ansible Automation Hub API
generated: '2026-08-29'
method: generated
source: openapi/red-hat-ansible-automation-platform-automation-hub-openapi.json
operations:
  - api_automation_hub_content_v3_plugin_ansible_content_collections_index_list
  - api_automation_hub_content_v3_plugin_ansible_content_collections_index_versions_list
  - api_automation_hub_content_v3_plugin_ansible_imports_collections_read
  - api_automation_hub_content_v3_collections_versions_move_move_content
  - api_automation_hub_content_v3_collections_versions_copy_copy_content
  - _api_automation-hub_pulp_api_v3_tasks_tasks_list
  - _api_automation-hub_pulp_api_v3_tasks_{pulp_id}_tasks_read
  - tasks_cancel
---

# Publish and certify collections in Automation Hub

Two bases, same contract:

- Red Hat-hosted: `https://console.redhat.com/api/automation-hub/`
- Private hub on a customer install: `https://<aap-gateway-host>/api/galaxy/`

Authenticate with an Automation Hub token (`Authorization: Bearer`) or HTTP Basic.

## Pagination is different here

Automation Hub is a Pulp application. It uses `limit` + `offset`, and returns
`meta.count` plus `links.first/previous/next/last` — **not** the `page`/`page_size` +
`count/next/previous/results` envelope the rest of AAP uses. Do not carry Controller paging code
across.

## Use the /plugin/ansible/ paths

The legacy `/content/v3/collections/` and `/v3/collections/` surfaces are deprecated — 53
operations in this spec carry `deprecated: true`. Prefer
`/content/v3/plugin/ansible/content/collections/index/`.

## Find content

`GET .../plugin/ansible/content/collections/index/` then
`.../index/{namespace}/{name}/versions/` for versions.

## Publish

Publishing is normally done with `ansible-galaxy collection publish`, which posts the built
tarball to the imports endpoint. Track the import with
`api_automation_hub_content_v3_plugin_ansible_imports_collections_read`.

## Everything asynchronous returns a task

153 operations in this spec return `202 Accepted` with a task href. Poll the task
(`_api_automation-hub_pulp_api_v3_tasks_{pulp_id}_tasks_read`, GET
`/api/automation-hub/pulp/api/v3/tasks/{pulp_id}/`) until the state leaves `waiting`/`running`;
list with `_api_automation-hub_pulp_api_v3_tasks_tasks_list`. A task that has not completed can
be stopped with `tasks_cancel` (PATCH the task) — that is the reversal window, and it closes as
soon as the task completes.

## Certification is a move, not a flag

Approving a collection version is modelled as moving it between repositories:
`api_automation_hub_content_v3_collections_versions_move_move_content` (and `_copy_copy_content`
to copy instead). This is reversible — move it back — which makes it one of the few genuinely
undoable writes in the platform.

Deleting a published collection version is **not** reversible and no restore window is
documented.
