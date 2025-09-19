# Showcase of various DevOps related tools

This repository is designed to be a showcase for sharing different methods of implementing some useful DevOps tools and scripts.

The first showcase, [Proxmox VE](docs/PVE.md), will upload image files to each of the PVE nodes `local` storagefrom the `files` directory. Because they can be large files, the `files` directory is in `.gitignore` and not saved in the repository. The `acquire.yml` playbook can be used to populate this directory after cloning this repository and before working on any of the showcase items. Uncomment or add any image files as desired.

```bash
ansible-playbook acquire.yml
```

## Showcased Items

- [Proxmox VE](docs/PVE.md), cluster the Proxmox VE nodes and configure some items.
- [Build](docs/Build.md), install packages inside cloud init image files.
- [Ceph Rados Gateway](docs/Ceph-RGW.md), configure Ceph and Ceph RadosGW.
- [Extra](docs/Extra.md), using `cicustom` for `vendor_data` (think Cloud Init `user_data`).
- [Garage](docs/Garage.md), the `apt-cacher` and `netboot_xyz` services.
- [Grafana Microservices Stack](docs/Grafana_Microservices_Stack.md/), in `k0s`.
- [Grafana Monolithic Stack](docs/Grafana_Monolithic_Stack.md), Monolithic Stack, in VMs.
- [Known Hosts](docs/Known_Hosts.md), utility playbook to update `~/.ssh/known_hosts` on the control node.
- [Kubernetes_k0s](docs/Kubernetes_k0s.md), create a Kubernetes cluster using `k0s`.
- [Name Server](docs/Name_Server.md), multiple playbooks demonstrating different DNS Name Server deployments.
- [Reverse Proxy](docs/Reverse_Proxy.md), create a couple of CTs and configure them to be NGINX reverse proxy servers. Or, to set up a server for Ubuntu autoinstall.
- [Study](docs/Study.md), VM for study of new concepts and applications.
- [Umbrella](docs/Umbrella.md), another VM for study of new concepts and applications.

# Details

For reproducability by others, these instructions are written for Ubuntu. This could be a computer with Ubuntu Desktop, or a VM running under one of:

- [Canonical Multipass](https://multipass.run/), recommended because it supports Mac and Windows
- [Parallels](https://www.parallels.com/)
- [Proxmox Virtual Environment](https://www.proxmox.com/en/proxmox-virtual-environment/overview)
- [VMware](https://www.vmware.com/)

This repository has be written and tested using Multipass on a Mac. Caution, MacOS Sequoia has a potential limitation due to a new "feature" that limits access to the Local Network (under Privacy & Security) to specific applications. Figuring out how to get around the issues created by this new feature is beyond the scope of this showcase and will hopefully no longer be an issue by the time you read this.

# Assumptions / Prerequisites

Most Ansible playbooks require one or more of the roles in the collections listed in `requirements.yml` which must be installed on the Ansible control host.

```bash
ansible-galaxy collection install -r requirements.yml
```

All showcase items have been designed to work for a test lab located in a dedicated LAN or VLAN with the following attributes.

- Network CIDR block `192.168.6.0/24` (netmask `255.255.255.0`).
- Default route of `192.168.6.1`.
- DNS configured with various host names that resolve to IP addresses (See the `zone_vars/lab.yml` and `zone_vars/192.168.6.yml` files).
- Three physical systems capable of running the [Proxmox Virtual Environment](https://www.proxmox.com/en/proxmox-virtual-environment/overview).
- A computer or VM to be the [Ansible control host](#ansible-control-host-setup) (where you run Ansible).
- A clone of [this repository](https://github.com/cloudcodger/showcase.git).
  - For the [Multipass setup](#multipass-setup), this is cloned to the localsystem and mounted in the VM at `~/showcase`.

## VMID Numbering Convention

For each CT or VM, the convention is to use the IP address as the basis for the VMID. This means that they all start with `6` (the third octet) followed by a three digit number (the forth octet). For example, the IP `192.168.6.8` would be configured to use the VMID of `6008`. For guest systems that use DHCP, the VMID is set to the next available number (starting at `100`) that Proxmox VE finds available.

## Domain name

The domain name for this network is set using the `showcase_base_domain` variable, with a default of `example.com`. This is prepended with `lab` (default `lab.example.com`). In order to use a different base domain (maybe one you have registered), set the environment varialbe `SHOWCASE_BASE_DOMAIN` or change the value in [`all.yml`](lab/group_vars/all.yml). Setting `SHOWCASE_BASE_DOMAIN` allows the repository to be used as is and setting and exporting it in `~/.bashrc` will make sure it's set for every shell.

# Ansible control host setup

Connect to the computer that will be used as your Ansible control node and clone this repository. See the [Multipass setup](#multipass-setup) option or the [Mac estup](#mac-setup) option.

## Multipass setup

Commands to create an Ubuntu VM named `showcase` on MacOS and mount the repository inside it.

```bash
REPO_DIR="${HOME}/git-repos/cloudcodger/showcase"
git clone git@github.com:cloudcodger/showcase.git ${REPO_DIR}
brew install --cask multipass
multipass launch -c 4 -d 20G -m 4G -n showcase --mount ${REPO_DIR}:/home/ubuntu/showcase
multipass shell showcase
```

The `--mount` allows the use of [VSCodium](https://vscodium.com/) on the Mac for editing, while still running commands inside the virtual machine. If you just want to run commands and are OK using something like `vim` for editing the files, you can always just clone the repository inside the VM.

For example, clone this git repository to `~/git-repos/cloudcodger/showcase` (done by the `multipass launch` parameter `--mount`).

This repository could be cloned to a different location. In which case, modify the path for the `vminit` shell script and the `cd` command with the correct path.

```bash
showcase/bin/vminit
cd showcase
```

## Mac setup

Some issues have come up with the latest Python3 when it comes to `proxmoxer`. For this reason, the use of Python3 version 3.12.9 my be required. The Ansible built in collection `community.general` on the Mac has also show problems and had to be downgraded to version `9.5.0`.

Commands to run directly on MacOS.

```bash
REPO_DIR="${HOME}/git-repos/cloudcodger/showcase"
git clone git@github.com:cloudcodger/showcase.git ${REPO_DIR}

brew install pyenv
brew install pyenv-virtualenv
echo 'eval "$(pyenv init -)"' >> ~/.zshrc
echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.zshrc
source ~/.zshrc
pyenv install 3.12
pyenv virtualenv 3.12 venv12
pyenv activate venv12 # This can be put in ~/.zshrc as well, if desired

cd ${REPO_DIR}
pip3 install -r python-requirements.txt
ansible-galaxy collection install community.general==9.5.0 # may not need
ansible-galaxy collection install -r requirements.yml
```

# Ansible Inventory from Proxmox VE


## Inventory Source

The inventory for the showcase has been [organized into a directory](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html#organizing-inventory-in-a-directory). This provides the ability to combine dynamic and static inventory sources.

## Dynamic Proxmox Inventory

The playbooks in this repository take advantage of a [Proxmox inventory source](https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_inventory.html) to dynamically build the inventory from the PVE cluster.

The `lab/inventory.proxmox.yml` file name ends with `.proxmox.yml` as required in the above link.

When the cluster is first created, the `url: https://192.168.6.251:8006/` will use a self signed certificate and `validate_certs: false` must be set for it to work, unless you first configure SSL certificates with something like [Let's Encrypt](https://pve.proxmox.com/pve-docs/pve-admin-guide.html#sysadmin_certs_get_trusted_acme_cert).

The `keyed_groups:` section will create an Ansible group for each tag on the CTs and VMs. This allows a playbook to create a set of VM with a specific tag that a subsequent playbook can reference in the `hosts:` line, so it will only run on the desired set of hosts and not all the hosts in the PVE cluster.
The `compose:` section will set `ansible_host` to the IP address of the guest machine, but cannot get the IP address of a VM using DHCP. This is where the static inventory files are needed.

Also notice that the `token_secret` is read from the file `~/.pve_tokens/lab-s1m0ne-pve-ansible.token`. The correct token must be located in this file and is created the first time the `pve.yml` playbook is run. When using multiple Ansible control nodes with this repository, keeping a copy of this token updated across all the control nodes is outside the scope of this readme. If you loose the token, it can be recreated by using the PVE UI to delete the API token and running the `pve.yml` playbook again to create a new one.
