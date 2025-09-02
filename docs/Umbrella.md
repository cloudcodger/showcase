# Creating a Generic Ubuntu machine

This showcase demonstrates how to create a QEMU VM for any kind of testing you want. Can be very handy when following along with a tutorial for a new service being studied.

Like the [Study](Study.md) showcase, but primarily for Ubuntu. Although it can also be used when you need two VMs for working on a tutorial or test setup.

Quick commands:

```bash
ansible-playbook -i lab umbrella.yml
# can be destroyed with
ansible-playbook -i lab pve_remove_guests.yml -e host_list=umbrella
```

# Playbooks

## `umbrella.yml`

Creates the QEMU VM `umbrella`.

```bash
ansible-playbook -i lab umbrella.yml
```
