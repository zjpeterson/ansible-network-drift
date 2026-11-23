# ansible-network-drift

Ansible playbooks that detect configuration drift on network devices and fix it through a ServiceNow change request workflow.

The desired state of each device is defined as data in `group_vars` and `host_vars`. The [`ansible.netcommon.network_resource`](https://docs.ansible.com/ansible/latest/collections/ansible/netcommon/network_resource_module.html) module compares that data to each device's running config. The same code works across vendors (Cisco IOS, Arista EOS, Juniper Junos).

## What it does

- **`detect_drift.yml`**: finds devices whose config doesn't match the desired state. For each one, it opens a ServiceNow change request that lists the commands needed to correct it.
- **`resolve_drift.yml`**: applies the corrections for devices with an approved drift change. It records what was applied on the change and closes it.

The ServiceNow work is done by the `snow_change` role from the [`zjpeterson.labcommon`](https://github.com/zjpeterson/ansible-labcommon) collection.

## Managed config

| Resource      | Variable          |
|---------------|-------------------|
| `snmp_server` | `net_snmp_server` |
| `interfaces`  | `net_interfaces`  |

This is used for demo purposes and intentionally doesn't have broad coverage of possible network resources. To manage additional resources, add an entry to `config_mgmt_network_resources` in `roles/config_mgmt/vars/main.yml` and define its variable in `group_vars`/`host_vars`.

## Requirements

- Ansible (ansible-core)
- Collections from `collections/requirements.yml`:
  - `ansible.network`
  - `zjpeterson.labcommon`
  - `servicenow.itsm`
- A ServiceNow instance. Credentials are read from the standard `servicenow.itsm` environment variables (`SN_HOST`, `SN_USERNAME`, `SN_PASSWORD`).

```bash
ansible-galaxy collection install -r collections/requirements.yml -p collections
```

## Usage

`inventory_example` and `inventory_containerlab.yml` are example inventories.

```bash
export SN_HOST=https://instance.service-now.com
export SN_USERNAME=...
export SN_PASSWORD=...

# Find drift and open change requests
ansible-playbook -i <inventory> detect_drift.yml

# Once the change is approved and scheduled, fix the drift
ansible-playbook -i <inventory> resolve_drift.yml
```
