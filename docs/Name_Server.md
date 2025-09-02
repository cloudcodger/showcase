# Installing and Configuring Name Servers

This showcase includes multiple methods of creating and configuring name servers using DNS Bind9.

Play around with it by commenting out different plays to see the desired configuration(s).

The `name_servers.yml` playbook contains multiple plays. They are:

## 1. Create all Bind9 Name Servers. CTs and VMs.

Creates the LXC containers `ns0` thru `ns6` and the QEMU VMs `ns7` thru `ns9`.

Because the variable names differ between LXC and QEMU, all are set in the [local_name_server](../lab/host_vars/local_name_server.yml) file.

## 2. Standard Bind9 Servers. One primary and two secondary.

Configures a standard Bind9 Name Server setup in the CTs tagged with `name_server`.

The standard configuration here consists of one Primary name server with multiple Secondary names servers that reference it. Many will debate if this really is a standard configuration.

The configuration uses [name_server.yml](../lab/group_vars/name_server.yml) to set variables for the Secondary name servers and [ns1.yml](../lab/host_vars/ns1.yml) to override those variables to configure that host as the Primary name server.

## 3. Single Bind9 Name Server.

Configure a single CT as a Bind9 Name Server where no redundancy is required.

The configuration uses [ns_single.yml](../lab/group_vars/ns_single.yml) to set the variables.

## 4. Split Bind9 Name Servers.

Configure three CTs. The first two are configured as a Primary name server for one domain and a Secondary name server for another. Each pointing to the other as the secondary name servers primary. The third CT is configured as a Secondary name server for both domains and an example of how to configure a forwarded zone. This demonstrates how flexible the `cloudcodger.dns.bind9_servers` role is.

The configuration uses [ns_split.yml](../lab/group_vars/ns_split.yml), only to set the `bind9_servers_zone_default_ttl` variable. Since every host has a unique configuration, they are each configured using [ns4.yml](../lab/host_vars/ns4.yml), [ns5.yml](../lab/host_vars/ns5.yml), and [ns6.yml](../lab/host_vars/ns6.yml).

## 5. Standard Bind9 Servers on VMs. One primary and two secondary.

Configuration the same as #2 above that exists simply to showcase that the `cloudcodger.dns.bind9_servers` role can be used for either CTs or VMs.

The configuration uses [ns_vm.yml](../lab/group_vars/ns_vm.yml) to set variables for the Secondary name servers and [ns7.yml](../lab/host_vars/ns7.yml) to override those variables to configure that host as the Primary name server.

Quick commands:

```bash
ansible-playbook -i lab name_servers.yml
# can be destroyed with
ansible-playbook -i lab pve_remove_guests.yml -e host_list=name_server,ns_single,ns_split,ns_vm
```

# Proxmox tags

The [Proxmox Inventory source](https://docs.ansible.com/ansible/latest/collections/community/general/proxmox_inventory.html) `lab` is configured to provide Ansible groups from the Tags on each CT and VM.

This showcase creates the following Ansible groups (from tags).

- [`name_server`](../lab/group_vars/name_server.yml)
  - A group of containers with one Primary name server and two Secondary name servers.
- [`ns_single`](../lab/group_vars/ns_single.yml)
  - A single standalone DNS server.
- [`ns_split`](../lab/group_vars/ns_split.yml)
  - Multiple individual primary DNS servers.
- [`ns_vm`](../lab/group_vars/ns_vm.yml)
  - Same as `name_server` only with VMs instead of LXC containers.

# Playbooks

## `name_servers.yml`

The playbook that does everything in this showcase.

```bash
ansible-playbook -i lab name_servers.yml
```
