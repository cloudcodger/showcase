# Creating a Build machine

This showcase creates an LXC container (CT) and uses it to install the `acl` and `qemu-guest-agent` packages and do the `--update` in `cloud-init` images. An example of how to pre-install packages into cloud-init images so they are immediately available when they first boot.

Features of the `cloudcodger.proxmox_client.lxc` role demonstrated in this playbook.

- `lxc_ansible_host` to create a `host_vars` file with an `ansible_host` variable.
- `lxc_ansible_inventory_refresh` so the next play can find `hosts: builder`.
- `lxc_cts` for just one CT with only a `name`. Making it use DHCP and auto select the VMID.

The playbook does not attempt to skip any of the steps in this process as running the playbook is only done when one of more files are to be updated.

Proxmox VE supports the QEMU Guest Agent for QEMU VMs which requires the VM to have the `qemu-guest-agent` package installed. When creating a large number of VMs in a Proxmox VE cluster, it will save time to pre-install this package into an image used for creating the VMs. When creating VMs that use DHCP to obtain an IP address and using the QEMU Guest Agent to obtain the IP address, it must already exist on the VM when it boots.

This playbook can either download the image from a URL or copy one from the `files/` directory (which is not committed to the git repository). It then installs the guest agent into the image file and downloads the resulting image to the `files/` directory.

The [Umbrella](Umbrella.md) showcase can be used to verify an Ubuntu image works. After creating the VM using one of the updated images, shut it down and select the VM in the PVE UI. Under Options, change the QEMU Guest Agent to enable it. Then start it again. The Guest Agent information will then appear on the Summary page.

Quick commands:

```bash
ansible-playbook -i lab builder.yml
# can be destroyed with
ansible-playbook -i lab pve_remove_guests.yml -e host_list=builder
```

# Playbooks

## `builder.yml`

Creates the LXC container `builder`. Make sure you allocate enough memory for `virt-customize` to use.

Configures the LXC container `builder` to install `libguestfs-tools`, upload or download cloud init image files, install the `acl` and `qemu-guest-agent` packages inside the images, and then download the resulting image files.

Before running the playbook, update the `images` variable in the `lab/host_vars/builder.yml` file to contain the `src` and `dest` for image files to be updated.

```bash
ansible-playbook -i lab builder.yml
```
