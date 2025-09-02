# Creating a Study machine

This showcase demonstrates how to create an LXC container or a QEMU VM for any kind of testing you want. Can be very handy when following along with a tutorial for a new service being studied.

This can also be used to study the different settings that can be provided to the `cloudcodger.proxmox_client.cloud_init` and `cloudcodger.proxmox_client.lxc` roles.

Quick commands:

```bash
ansible-playbook -i lab study.yml
# or
ansible-playbook -i lab study_vm.yml
# either can be destroyed with
ansible-playbook -i lab.inventory.proxmox.yml pve_remove_guests.yml -e host_list=study
```

# Playbooks

## `study.yml`

Creates the LXC container `study`.

```bash
ansible-playbook -i lab study.yml
```

## `study_vm.yml`

Creates the QEMU VM `study`.

```bash
ansible-playbook -i lab study_vm.yml
```
