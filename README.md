# Showcase of various DevOps related tools

This repository is designed to be a showcase for sharing different methods of implementing some useful DevOps tools and scripts.

For reproducability by others, these instructions are written for Ubuntu. This could be a computer with Ubuntu Desktop, or a VM running under one of:

- [Canonical Multipass](https://multipass.run/), recommended because it supports Mac and Windows
- [Parallels](https://www.parallels.com/)
- [Proxmox Virtual Environment](https://www.proxmox.com/en/proxmox-virtual-environment/overview)
- [VMware](https://www.vmware.com/)

This repository will be written and tested using Multipass on a Mac. Caution, MacOS Sequoia has a potential limitation due to a new "feature" that limits access to the Local Network (under Privacy & Security) to specific applications. Figuring out how to get around the issues created by this new feature is beyond the scope of this showcase.

Commands to create an Ubuntu VM named `showcase`.

```bash
brew install --cask vscodium
brew install multipass
multipass launch -c 4 -d 20G -m 4G -n showcase --mount ~/git-repos:/home/ubuntu/git-repos
multipass shell showcase
```

The `--mount` allows me to use [VSCodium](https://vscodium.com/) on the Mac and run commands inside the virtual machine in the same directory. If you just want to run commands and are OK using something like `vim` for editing the files, you can always just clone the repository inside the VM.

# Assumptions / Prerequisites

The various scripts have been created to work for a test lab located in a dedicated LAN or VLAN.

- Network CIDR block `192.168.6.0/24` (netmask `255.255.255.0`), with default route of `192.168.6.1` and DNS configured with various host names that resolve to IP addresses (See the `zone_vars/lab.yml` and `zone_vars/192.168.6.yml` files).
- Three physical systems capable of running the [Proxmox Virtual Environment](https://www.proxmox.com/en/proxmox-virtual-environment/overview).
- A computer or VM to be the Ansible control host (where you run Ansible).
- This git repository has been cloned to `~/git-repos/cloudcodger/showcase` (when using Multipass).

## Domain name

The domain name for this network is set using the `showcase_base_domain` variable, with a default of `example.com`. This is prepended with `lab` (default `lab.example.com`). In order to use another base domain (maybe one you have registered), set the environment varialbe `SHOWCASE_BASE_DOMAIN` and export it. Putting it in `~/.bashrc` will make sure you don't forget to set it.

# Ansible control node setup

Connect to the computer or VM that will be used as your Ansible control node (done by `multipass shell`).

Clone this git repository to `~/git-repos/cloudcodger/showcase` (done by the `multipass launch` parameter `--mount`).

This repository could be cloned to a different location. In which case, modify the path for the `vminit` shell script and the `cd` command with the correct path.

```bash
git-repos/cloudcodger/showcase/bin/vminit
source ${HOME}/.venv/bin/activate
cd git-repos/cloudcodger/showcase
```

# Showcased Items

- [Proxmox VE](docs/PVE.md), cluster the Proxmox VE nodes and configure some items.
- [Extra](docs/Extra.md), using `cicustom` for `vendor_data` (think Cloud Init `user_data`).
- [Garage](docs/Garage.md), the `apt-cacher` and `netboot_xyz` services.
- [Name Server](docs/NameServer.md), multiple playbooks demonstrating different DNS Name Server deployments.
- [Study](docs/Study.md), VM for study of new concepts and applications.
- [Umbrella](docs/Umbrella.md), another VM for study of new concepts and applications.
