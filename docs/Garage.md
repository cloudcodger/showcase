# Installing and Configuring Garage Services

This showcase demonstrates how to create an LXC container and install `apt-mirror` and `netboot_xyz`. It also makes use of `cloudcodger.ubuntu.initial_apt_update` to perform an `apt update; apt dist-upgrade` the first time the playbook is run since this is frequently the first thing done on a new container.

This also demonstrates how to configure a second virtual disk that gets mounted at `/apt-mirror` in the LXC container. Note the `mp0: "local-lvm:150,mp=/apt-mirror/"` line in the `custom_vars/garage.yml` file. The `local-lvm` must be a valid storage item in the cluster and the `150` is the size of this disk in GB. Make sure to adjust this according to the amount of data to be mirrored and the total size of your storage.

Quick commands:

```bash
ansible-playbook garage_container.yml

ansible-playbook -i lab.inventory.proxmox.yml garage.yml

ansible-playbook -i lab.inventory.proxmox.yml garage_destroy.yml
```

# Playbooks

## `garage_container.yml`

Creates the QEMU VM `garage`, using `custom_vars/garage.yml` (See, [custom_vars](Custom_vars.md)).

```bash
ansible-playbook garage_container.yml
```

## `garage.yml`

Configures the LXC container `garage` with multiple services.

1) An `apt-mirror` for use as a local APT repository when desired.
2) A NetBoot system, with `netboot_xyz` installed.
3) An NGINX web server.

```bash
ansible-playbook -i lab.inventory.proxmox.yml garage.yml
```

## `garage_destroy.yml`

Destroys the `garage` container and cleans up SSH keys.

```bash
ansible-playbook -i lab.inventory.proxmox.yml garage_destroy.yml
```
