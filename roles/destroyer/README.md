Role `destroyer`
================

This role is a quick way to destroy all the PVE Guests and remove the SSH keys from `~/.ssh/known_hosts` for the `destroyer_group`.

When creating and destroying a group of LXC containers or VMs over and over for testing things. This allows a quick and easy way to clean up a group of guest systems in order to re-test from a clean PVE cluster.

Requirements
------------

A Proxmox PVE environtment containing the guest systems in the Ansible group `destroyer_group`.

Role Variables
--------------

- `destroyer_group` - The ansible group of guest systems to destroy (required, no default).

Dependencies
------------

The `community.general.proxmox` and `community.general.proxmox_kvm` modules and their requirements (for example, the python `proxmoxer` module).

Example Playbook
----------------

```bash
---
- name: Example of Stoping and Removing 'study' VMs to clean up after testing.
  gather_facts: false
  hosts: localhost

  roles:
    - role: destroyer
      destroyer_group: study_vm
```

```bash
---
- name: Example of Stoping and Removing 'study' LXC containers to clean up after testing.
  gather_facts: false
  hosts: localhost

  roles:
    - role: destroyer
      destroyer_group: study_ct
```

```bash
---
- name: Example of Stoping and Removing 'study' LXC containers and VMs to clean up after testing.
  gather_facts: false
  hosts: localhost

  roles:
    - role: destroyer
      destroyer_group: study_ct
    - role: destroyer
      destroyer_group: study_vm
```
