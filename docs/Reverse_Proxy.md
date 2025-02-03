# Configuring NGINX for Reverse Proxy

This showcase demonstrates the use of the `cloudcodger.ubuntu.keepalived` and `cloudcodger.ubuntu.nginx` roles to create two LXC containers and configure them to be reverse proxy servers. It includes the installation of `keepalived` to add redundency for accessing the contianers with a VIP that automatically moves from the primary to the secondary when the primary becomes unavailable.

Two systems (`rotary[12]`) are configured with NGINX and using a third IP address (`rotary`) as the VIP in an active standby configuration. In case one system goes down, the other will take over the VIP and continue working. Providing fault tolerance for the load balancers.

The load balancers configured in this showcase are for Grafana Loki, Grafana Mimir, MinIO, and the MinIO UI. To see them work, the other components must be created as well. See, [MinIO](MinIO.md) and [Grafa Monolithic Stack](Grafana_Monolithic_Stack.md). This is not used for Grafana Loki or Grafana Mimir for the microservices approach.

The showcase also demonstrates how you can use the `nginx` role to create sites to serve up `meta-data` and `user-data` files for Ubuntu autoinstall. Details of how to do this are left to the reader as an exorise, since the use of `cloudcodger.proxmox_client.cloud_init` and [Extra](Extra.md) for custom configurations on PVE is now the prefered method of creating VMs.

Quick commands:

```bash
ansible-playbook rotary_container.yml
ansible-playbook -i lab.inventory.proxmox.yml rotary.yml
# can be destroyed with
ansible-playbook -i lab.inventory.proxmox.yml rotary_destroy.yml
```

# Playbooks

## `rotary_container.yml`

Creates the `rotary` LXC containers with the custom variables set directly in the playbook.

```bash
ansible-playbook rotary_container.yml
```

## `rotary.yml`

Configures the `rotary` LXC containers.

```bash
ansible-playbook -i lab.inventory.proxmox.yml rotary.yml
```

## `rotary_destroy.yml`

Destroys the `rotary` containers and cleans up SSH keys.

```bash
ansible-playbook -i lab.inventory.proxmox.yml rotary_destroy.yml
```
