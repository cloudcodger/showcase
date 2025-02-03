# Creating a machine with a Custom Configuration

This showcase demonstrates how to create a QEMU VM that uses the [`cicustom`](https://pve.proxmox.com/wiki/Cloud-Init_Support#_custom_cloud_init_configuration).

A typical need with cloud-init is the need to provide custom `user-data`. Because Proxmox internally provides `user-data` for various things, providing your own would override the builtin settings, leaving the end user to make sure and include them in the custom `user-data`. This adds complexity and confusion for anyone looking at the UI that would contain values that were overridden. This can be avoided. Using the `vendor-data` allows for the same setup items and is not used by Proxmox. This showcase demonstrates using it to modify the default APT repository for a system such that it would use the system created in the [extra](extra.md) showcase.

Quick commands:

```bash
ansible-playbook -i pve_hosts.ini extra_snippet.yml

ansible-playbook extra_vm.yml
# can be destroyed with
ansible-playbook -i lab.inventory.proxmox.yml extra_destroy.yml
```

# Requirements

Proxmox VE Datacenter storage `configs`, with `snippets` content allowed.

# Playbooks

## `extra_snippet.yml`

This first creates a snippet on the Proxmox VE cluster using the `cloudcodger.proxmox_openssh.proxmox_snippet` role. This requires one of the settings performed by the `pve.yml` playbook inside the `cloudcodger.proxmox_openssh.proxmox_datacenter` role. It creates the `configs` storage for `snippets` content under the Corosync file system. Check out the `README.md` file for that role to understand the limitations of the `configs` storage. This will contain the `vendor-data.yml` snippet created by this playbook and used by the QEMU VM.

Note that this is only being run on one of the PVE hosts because Corosync will sync it across the other cluster nodes. And updates a PVE nodes and not the host `extra`.

```bash
ansible-playbook -i pve_hosts.ini extra_snippet.yml
```

## `extra_vm.yml`

Create the `extra` QEMU VM using the `vendor_data.yml` snippet.

By logging into the Proxmox VE UI before executing the playbook, once the VM is created, select it and bring up the console when it becomes started. This is show how `cloud-init` will now use the APT URI set in the snippet. Check out the [cloud config examples](https://cloudinit.readthedocs.io/en/latest/reference/examples.html) for ideas on what can be set using `vendor_data`.

```bash
ansible-playbook extra_vm.yml
```

## `extra_destroy.yml`

Destroys the `extra` QEMU VM and cleans up SSH keys.

```bash
ansible-playbook -i lab.inventory.proxmox.yml extra_destroy.yml
```
