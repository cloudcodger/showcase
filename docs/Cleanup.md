# Cleaning up the lab cluster

To remove PVE Guest machines from the cluster, use the `pve_remove_guests.yml` playbook.

Just set `host_list` to a comma separated list of hosts and groups to remove all the guests.

Quick commands:

```bash
# Default is to remove the study and umbrella CTs and VMs
ansible-playbook -i lab pve_remove_guests.yml

# To remove ALL guest machines
ansible-playbook -i lab pve_remove_guests.yml -e host_list=proxmox_all_lcx,proxmox_all_qemu

# To remove specific guests or groups
ansible-playbook -i lab pve_remove_guests.yml -e host_list=builder
ansible-playbook -i lab pve_remove_guests.yml -e host_list=extra
ansible-playbook -i lab pve_remove_guests.yml -e host_list=garage
ansible-playbook -i lab pve_remove_guests.yml -e host_list=k0s
ansible-playbook -i lab pve_remove_guests.yml -e host_list=mimir
ansible-playbook -i lab pve_remove_guests.yml -e host_list=minio,miniolb
ansible-playbook -i lab pve_remove_guests.yml -e host_list=name_server,ns_single,ns_split,ns_vm
ansible-playbook -i lab pve_remove_guests.yml -e host_list=rotary
ansible-playbook -i lab pve_remove_guests.yml -e host_list=study
ansible-playbook -i lab pve_remove_guests.yml -e host_list=umbrella
ansible-playbook -i lab pve_remove_guests.yml -e host_list=watchdog
```
