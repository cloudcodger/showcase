# Proxmox Dynamic Inventory script

The playbooks in this repository take advantage of a [Proxmox inventory source](https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_inventory.html) to dynamically build the inventory from Proxmox VE.


# Inventory Source Configuration File

The `lab.inventory.proxmox.yml` file is the source configuration file used for this showcase. The file name ends with `.proxmox.yml` as required in the above link.

When the cluster is first created, the `url: https://192.168.6.251:8006/` will use a self signed certificate and `validate_certs: false` must be set for it to work.

The settings configured will create an Ansible group for each tag on the LXC containers and VMs. This allows a playbook to create a set of VM with a specific tag that a subsequent playbook can reference in the `hosts:` line, so it will only run on the desired set of hosts and not all the hosts in the Proxmox cluster.

Also notice that the `token_secret` is read from the file `~/.pve_tokens/lab-s1m0ne-pve-ansible.token`. The correct token must be located in this file and is created the first time the `pve.yml` playbook is run. When using multiple Ansible control nodes with this repository, keeping a copy of this token updated across those nodes is outside the scope of this repository. If you loose the token, it can be recreated by using the Proxmox UI to delete the API token and running the `pve.yml` playbook again to create a new one.
