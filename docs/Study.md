# Creating a Study machine

This showcase demonstrates how to create a QEMU VM for any kind of testing you want. Can be very handy when following along with a tutorial for a new service being studied.

This can also be used to study the different settings that can be provided to `cloudcodger.proxmox_client.cloud_init` and showcases the approach of providing the variables as parameters to the role instead of setting them in a `vars_files:` as shown in other showcase items.

Quick commands:

```bash
ansible-playbook study_container.yml
# or
ansible-playbook study_vm.yml
# can be destroyed with
ansible-playbook -i lab.inventory.proxmox.yml study_destroy.yml
```

# Playbooks

## `study_container.yml`

Creates the LXC container `study`.

```bash
ansible-playbook study_container.yml
```

## `study_vm.yml`

Creates the QEMU VM `study`.

```bash
ansible-playbook study_vm.yml
```

## `study_destroy.yml`

Destroys the `study` LXC container or VM and cleans up SSH keys.

```bash
ansible-playbook -i lab.inventory.proxmox.yml study_destroy.yml
```
