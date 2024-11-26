# Creating a Build machine

This showcase creates an LXC container and uses it to install the `qemu-guest-agent` package into `cloud-init` images.

Proxmox VE supports the QEMU Guest Agent for QEMU VMs which requires the VM to have the `qemu-guest-agent` package installed. When creating a large number of VMs in a Proxmox VE cluster, it will save time to pre-install this package into an image used for creating the VMs. This playbook can either download the image from a URL or copy one from the `files/` directory (which is not committed to the git repository). It then installs the guest agent into the image file and downloads the resulting image to the `files/` directory.

The [Umbrella](Umbrella.md) showcase can be used to verify an Ubuntu image works. After creating the VM using one of the updated images, shut it down and select the VM in the PVE UI. Under Options, change the QEMU Guest Agent to enable it. Then start it again. The Guest Agent information will then appear on the Summary page.

Quick commands:

```bash
ansible-playbook build_container.yml

ansible-playbook -i lab.inventory.proxmox.yml build.yml

ansible-playbook -i lab.inventory.proxmox.yml build_destroy.yml
```

# Playbooks

## `build_container.yml`

Creates the LXC container `build`. Make sure you allocate enough memory so that `virt-customize` has enough.

```bash
ansible-playbook build_container.yml
```

## `build.yml`

Configures the LXC container `build` to install `libguestfs-tools`, upload or download cloud init image files, install the `qemu-guest-agent` package inside the images, and then download the resulting image files.

Before running the playbook, update the `images` variable in the `build.yml` playbook file to contain those for which you want to install the guest agent.

```bash
ansible-playbook -i lab.inventory.proxmox.yml build.yml
```

## `build_destroy.yml`

Destroys the `build` container and cleans up SSH keys.

```bash
ansible-playbook -i lab.inventory.proxmox.yml build_destroy.yml
```
