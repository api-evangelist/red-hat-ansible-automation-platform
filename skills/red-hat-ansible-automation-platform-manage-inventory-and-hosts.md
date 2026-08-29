---
name: manage-ansible-inventory-and-hosts
description: >-
  Read and maintain the inventory Red Hat Ansible Automation Platform runs automation against —
  inventories, groups, hosts, host variables and dynamic inventory sources.
api: Red Hat Ansible Automation Controller API
generated: '2026-08-29'
method: generated
source: openapi/red-hat-ansible-automation-platform-automation-controller-openapi.json
operations:
  - inventories_list
  - inventories_create
  - hosts_list
  - hosts_create
  - hosts_variable_data_retrieve
  - groups_list
  - groups_create
  - inventory_sources_list
  - inventory_sources_create
  - inventory_sources_update_create
  - inventory_updates_cancel_create
  - hosts_ansible_facts_retrieve
---

# Manage inventory and hosts

Base path: `https://<aap-gateway-host>/api/controller/v2/`.

## Model

`Organization` owns `Inventory`; `Inventory` owns `Group` and `Host`; `InventorySource` is a
dynamic feed that populates an inventory from a cloud provider, SCM or script. See
`data-model/`.

## Read

- `GET /inventories/` (`inventories_list`) — `?search=` and `?order_by=name` work here as on
  nearly every Controller collection.
- `GET /hosts/` (`hosts_list`) — filter by `?inventory=<id>`; Django-style lookups are available
  (`?name__icontains=web`).
- `GET /hosts/{id}/variable_data/` (`hosts_variable_data_retrieve`) — the host's variables.
- `GET /hosts/{id}/ansible_facts/` (`hosts_ansible_facts_retrieve`) — facts gathered by the last
  run, if fact caching is on.

Every object carries a `related` map of URLs. Follow those rather than constructing paths.

## Write

- `POST /inventories/` (`inventories_create`) — requires `name` and `organization`.
- `POST /groups/` (`groups_create`) and `POST /hosts/` (`hosts_create`) — require `inventory`.
- Controller enforces uniqueness on (name, inventory) for hosts and groups, so a repeated create
  collides rather than duplicating. That is a data-model property, not an idempotency guarantee.

## Sync a dynamic source

`POST /inventory_sources/{id}/update/` (`inventory_sources_update_create`) starts an inventory
update job. It is cancellable while running:
`POST /inventory_updates/{id}/cancel/` (`inventory_updates_cancel_create`).

## Careful

Hosts are the unit AAP is licensed on. Bulk-creating hosts changes what the customer owes.
Treat `hosts_create` as a commercially consequential write.
