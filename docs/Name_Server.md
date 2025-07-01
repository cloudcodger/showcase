# Installing and Configuring Name Servers

This showcase includes multiple methods of creating and configuring name servers using DNS Bind9.

The `name_servers.yml`, `name_servers_single.yml`, and `name_servers_split.yml` playbooks all configure name servers inside of LXC containers created by the `name_server_containers.yml` playbook. The `name_servers_standard.yml` playbook configures servers inside VMs in the same way as `name_servers.yml` does. This is simply to make sure that the role can be used for either.

Each playbook will configure a different set of hosts.

Quick commands:

```bash
ansible-playbook name_server_containers.yml
ansible-playbook name_server_vms.yml
# or combined
ansible-playbook name_server_containers.yml  name_server_vms.yml

ansible-playbook -i lab.inventory.proxmox.yml name_servers.yml
ansible-playbook -i lab.inventory.proxmox.yml name_servers_single.yml
ansible-playbook -i lab.inventory.proxmox.yml name_servers_split.yml
ansible-playbook -i lab.inventory.proxmox.yml name_servers_standard.yml
# can be destroyed with
ansible-playbook -i lab.inventory.proxmox.yml pve_remove_guests.yml -e host_list=name_server,ns_single,ns_split,ns_vm
```

# Name Server Standard Setup

Both the `name_servers.yml` and `name_servers_standard.yml` playbooks configure 3 hosts such that one is a Primary and the other two are Secondary name servers. 

# Proxmox tags

The [Proxmox Inventory source](https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_inventory.html) `lab.inventory.proxmox.yml` is configured to provide Ansible groups from the Tags on each LXC or QEMU.

This showcase creates the following Ansible groups (from tags).

- `ns_single` - A single standalone DNS server.
- `name_server` - A group of containers with one Primary name server and two Secondary name servers.
- `ns_split` - Multiple individual primary DNS servers.
- `ns_vm` - Same as `name_server` only VMs instead of LXC containers.

# Playbooks

## `name_server_containers.yml`

Creates the LXC containers `ns0` thru `ns6`, using `custom_vars/name_server.yml` (See, [custom_vars](Custom_vars.md)). Because the variable names differ between LXC and QEMU, the same custom vars is used as the next playbook.

```bash
ansible-playbook name_server_containers.yml
```

## `name_server_vms.yml`

Creates the QEMU VMs `ns7` thru `ns9`, using `custom_vars/name_server.yml` (See, [custom_vars](Custom_vars.md)). Because the variable names differ between LXC and QEMU, the same custom vars is used as the previous playbook.

```bash
ansible-playbook name_server_vms.yml
```

## `name_servers.yml`

Configures the LXC containers `ns1` thru `ns3` as a single Primary name server using `host_vars/ns1.yml` and two Secondary name servers using `group_vars/name_server.yml`. Using the `host_vars/ns1.yml` allows that one to be configured differently than the others in the group. This showcase only includes two Secondary name server, but could have many more hosts in the Ansible group that would all be configured the same, for as many as desired.

```bash
ansible-playbook -i lab.inventory.proxmox.yml name_servers.yml
```

## `name_servers_single.yml`

Configures the LXC container `ns0` as a single standalone DNS server using `group_vars/ns_single.yml`. Because there is only one host in this group, these could just as easily be set in `host_vars/ns0.yml` and produce the same results.

```bash
ansible-playbook -i lab.inventory.proxmox.yml name_servers_single.yml
```

## `name_servers_split.yml`

Configures the LXC containers `ns4` thru `ns6` as individual name servers with special split configurations. One (`ns4`) is configured as the Primary for the `lab.{{ showcase_base_domain }}` zone and as Secondary for the `6.168.192.in-addr.arpa` zone. Another (`ns5`) is configured as Primary for `6.168.192.in-addr.arpa` and Secondary for `lab.{{ showcase_base_domain }}`. And a third (`ns6`) is configured as a Secondary pointing to different Primaries for each zone. Along with an example of how to configure a forwarder only for a specific zone.

```bash
ansible-playbook -i lab.inventory.proxmox.yml name_servers_split.yml
```

## `name_servers_standard.yml`

Configures the QEMU VMs `ns7` thru `ns9` as a single Primary name server using `host_vars/ns7.yml` and two Secondary name servers using `group_vars/ns_vm.yml`. Using the `host_vars/ns7.yml` allows that one to be configured differently than the others in the group. This showcase only differs from the above `name_servers.yml` above, in the `hosts:` are QEMU VMs instead of LXC containers. Because of this, the `ansible_user` is `ubuntu` for the Ubuntu QEMU VMs, where the LXC containers used `root` and did not need `become: true`.

```bash
ansible-playbook -i lab.inventory.proxmox.yml name_servers_standard.yml
```
