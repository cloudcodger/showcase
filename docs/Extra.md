# Creating a machine with a Custom Configuration

This showcase demonstrates how to create a QEMU VM that uses [`cicustom`](https://pve.proxmox.com/wiki/Cloud-Init_Support#_custom_cloud_init_configuration).

Features demonstrated in this playbook.

- The `cloudcodger.proxmox_openssh.proxmox_snippet` role to create a snippet.
- The `cloudcodger.proxmox_client.cloud_init` role using the snippet created.
- `cloud_init_ansible_host` to create a `host_vars` file with an `ansible_host` variable.
- `cloud_init_ansible_inventory_refresh` so the next play can find `hosts: builder`.
- `cloud_init_find_pm_host` to create the VM on the PVE node with the most free memory.
- `cloud_init_vms` with one VM using `custom`. Making it use DHCP and auto select the VMID.

Typical with cloud-init is the need to provide a custom `user-data`. Because Proxmox internally provides `user-data` for various things, providing your own would override the builtin settings, leaving the end user to make sure and include them in the custom `user-data`. This adds complexity and confusion for anyone looking at the UI that would contain values that were overridden. This can be avoided by using the `vendor-data`, which allows for the same setup items and is not used by Proxmox. This showcase demonstrates creating a [snippet](#snippetyml) that changes APT sources to use `garage` and creating a [VM](#extrayml) that uses it.

Quick commands:

```bash
ansible-playbook -i lab snippet.yml

ansible-playbook -i lab extra.yml
# can be destroyed with
ansible-playbook -i lab pve_remove_guests.yml -e host_list=extra
```

# Requirements

Proxmox VE Datacenter storage `configs`, with `snippets` content allowed.

# Playbooks

## `snippet.yml`

This first playbook creates a snippet on the Proxmox VE cluster. This requires one of the settings performed by the [`pve.yml`](PVE.md) playbook inside the `cloudcodger.proxmox_openssh.proxmox_datacenter` role. That playbook creates the `configs` storage for `snippets` content under the Corosync file system. Check out the `README.md` file for that role to understand the limitations of the `configs` storage. This will contain the `vendor-data.yml` snippet created by this playbook and used by the QEMU VM.

Note that this is only being run on one of the PVE hosts (`hosts: "{{ groups['proxmox_hosts'] | random }}"`) because Corosync will sync it over to the other PVE cluster nodes.

```bash
ansible-playbook -i pve_hosts.ini snippet.yml
```

## `extra.yml`

Create the `extra` QEMU VM using the `vendor_data.yml` snippet.

By logging into the Proxmox VE UI before executing the playbook, once the VM is created, select it and bring up the console when it becomes started. This will allow you to see that `cloud-init` will now use the APT URI set in the snippet. Check out the [cloud config examples](https://cloudinit.readthedocs.io/en/latest/reference/examples.html) for ideas on what can be set using `vendor_data`. Alternatively, login to the VM and `grep ^URI /etc/apt/sources.list.d/ubuntu.sources`.

```bash
ansible-playbook -i lab extra.yml
```
