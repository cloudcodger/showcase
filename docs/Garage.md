# Installing and Configuring Garage Services

This showcase demonstrates how to create an LXC container and install `apt-cacher` and `netboot_xyz`. It also makes use of `cloudcodger.ubuntu.initial_apt_update` to perform an `apt update; apt dist-upgrade` the first time the playbook is run since this is frequently the first thing done on a new container.

This also demonstrates how to configure a second virtual disk that gets mounted at `/apt-cacher` in the LXC container. Note the `mp0: "local-lvm:100,mp=/apt-cacher/"` line in the `custom_vars/garage.yml` file. The `local-lvm` must be a valid storage item in the cluster and the `100` is the size of this disk in GB. Make sure to adjust this according to the amount of data to be cached and the total size of your storage.

This originally showcased how to use [`apt-mirror`](https://github.com/apt-mirror/apt-mirror) with the `cloudcodger.ubuntu.apt_mirror` role. Because the `apt-mirror` project has not been maintained for many years, the role installed a patched version of the script that had been used up to Ubuntu 22.04. While adding the [Extra](Extra.md) showcase that requires a local repository, it was found that `apt-mirror` is once again not working. After many hours attempting to make it work with either Debian Bookworm or Ubuntu 24.04, the better approach of using `apt-cacher` was found and implemented instead.

Quick commands:

```bash
ansible-playbook garage_container.yml

ansible-playbook -i lab.inventory.proxmox.yml garage.yml

ansible-playbook -i lab.inventory.proxmox.yml garage_destroy.yml
```

# Playbooks

## `garage_container.yml`

Creates the LXC container `garage`, using `custom_vars/garage.yml` (See, [custom_vars](Custom_vars.md)).

```bash
ansible-playbook garage_container.yml
```

## `garage.yml`

Configures the LXC container `garage` with multiple services.

1) An `apt-cacher` for use as a local APT repository when desired.
2) A NetBoot system, with `netboot_xyz` installed.

```bash
ansible-playbook -i lab.inventory.proxmox.yml garage.yml
```

## `garage_destroy.yml`

Destroys the `garage` container and cleans up SSH keys.

```bash
ansible-playbook -i lab.inventory.proxmox.yml garage_destroy.yml
```
