# Update the `~/.ssh/known_hosts`

Loop over a `host_list` that is looked up in `inventory_hostnames` and run `ssh-keyscan` on each. Then update the `~/.ssh/known_hosts` with all non-comment lines.

For Proxmox VE guest systems that are created and removed frequently, the `~/.ssh/known_hosts` file on the control node may contain outdated entries. This can cause playbook execution to fail.

The playbook runs `ssh-keyscan` for both the `item` and the `ansible_host` value of the item. Then uses `lineinfile` to update the `~/.ssh/known_hosts` file. Because `ansible_host` may be set to the IP address of a host name (`item`) that does not exist in DNS, this will add both the name and the IP at the same time.

Default is to do this for the `proxmox_all_running` group. For a smaller group, set `group_name` or for a few hosts, set `host_list` to a comma seperated list of the names.

Quick commands:

```bash
ansible-playbook -i lab known_hosts.yml
ansible-playbook -i lab known_hosts.yml -e group_name=proxmox_hosts
ansible-playbook -i lab known_hosts.yml -e host_list=gadget1,gadget2,gadget3
```
