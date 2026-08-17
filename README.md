# Showcase of various DevOps related tools

This repository is a showcase for sharing different methods of implementing some useful DevOps tools and scripts. Primarily based around the [Proxmox Virtualization Environment(PVE)](https://www.proxmox.com/en/proxmox-virtual-environment) and both managing it and various applications running inside it using [Ansible](https://github.com/ansible/ansible).

The [Wiki](https://github.com/cloudcodger/showcase/wiki) pages provide details on how to use this repository.

First make sure that you have Python installed and all the required modules. The showcase is written and tested using Ansible as a PIP module and not an OS package. The [vminit](bin/vminit) shell script can be used on a freshly created Ubuntu VM to get the OS patched and a python virtual environment set up and ready for use as an [Ansible Control Node](https://github.com/cloudcodger/showcase/wiki/Ansible-Control-Node).

# Showcased Items

- [Launch A PVE Node](https://github.com/cloudcodger/showcase/wiki/PVE-Launch-Node).
- [Launch A PVE Cluster](https://github.com/cloudcodger/showcase/wiki/PVE-Launch-Cluster).


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

# Assumptions / Prerequisites

- A working [Ansible Control Node](https://github.com/cloudcodger/showcase/wiki/Ansible-Control-Node) with the required Python modules and Ansible collections installed.
- A LAN or VLAN for showcase use. This repository uses the following values:
  - Network CIDR block `192.168.6.0/24` (netmask `255.255.255.0`).
  - Default route of `192.168.6.1`.
  - Has DNS name resolution configured.
- One (1), three (3) or four (4) system(s) capable of running the [PVE](https://www.proxmox.com/en/proxmox-virtual-environment) software.
- A clone of [this repository](https://github.com/cloudcodger/showcase.git).

A different network CIDR block can be used but will require you to find and change it where set.

# Domain name

The domain name for the PVE network is set using the `showcase_base_domain` variable, with a default of `example.com`. This base domain is then prepended with `lab` (constructed default `lab.example.com`). In order to use a different base domain (maybe one you have registered), the preferred method is to set (and export) the environment variable `SHOWCASE_BASE_DOMAIN`. Setting and exporting `SHOWCASE_BASE_DOMAIN` in `~/.bashrc` (or `~/.zshrc` for macOS), allows the repository to be used as is and will make sure it's set for every spawned shell. The `showcase_base_domain` variable could also be changed in the `group_vars/all.yml` files.
