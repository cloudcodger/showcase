# Creating a Generic Ubuntu machine

This showcase demonstrates how to create a QEMU VM for any kind of testing you want. Can be very handy when following along with a tutorial for a new service being studied.

Like the [Study](Study.md) showcase, but primarily for Ubuntu. Although it can be also be used when you need two VMs for working on a tutorial or test setup.

It also showcases another approach of providing the variables, instead of setting them in a `vars_files:` as shown in other showcase items.

Quick commands:

```bash
ansible-playbook umbrella_vm.yml
# can be destroyed with
ansible-playbook -i lab.inventory.proxmox.yml pve_remove_guests.yml -e host_list=umbrella
```

# Playbooks

## `umbrella_vm.yml`

Creates the QEMU VM `umbrella`.

```bash
ansible-playbook umbrella_vm.yml
```
