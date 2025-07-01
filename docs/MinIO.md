# An S3 style Object Store.

A showcase for [MinIO](https://min.io/). A replacement for an AWS S3 bucket for [Grafana Mimir](https://grafana.com/oss/mimir/) and [Grafana Loki](https://grafana.com/oss/loki/). Consisting of three (3) VMs running [MinIO](https://min.io/) in a cluster and two (2) LXC containers running NGINX as a load balancer in front of the cluster.

Steps:

1. Create two (2) LXC containers (`miniolb[12]`) for NGINX (the `minio_containers` playbook).
2. Create three (3) of VMs (`minio[123]`) for the MinIO cluster (the `minio_vms.yml` playbook).
3. Create the file `~/.secrets/minio/root_password` with a string for the `minio_root_password`.
4. Configure the VMs and LXC containers (the `minio.yml` playbook).
    - Installs `nginx`, `keepalived` and `minio`.
    - Step 6 assumes that the `keepalived` VIP is mapped to `minio` in DNS.
5. [Login](http://minio/minio/ui/login) to the UI (as `admin` and the password in `~/.secrets/minio/root_password`).
6. Create an Access Key for use by other showcases.
    - Click on [Access Keys](http://minio/minio/ui/access-keys) on the left and then [Create Access Key](http://minio/minio/ui/access-keys/new-account).
    - Save the Access Key in `~/.secrets/minio/AWS_ACCESS_KEY_ID`
    - Save the Secret Key in `~/.secrets/minio/AWS_SECRET_ACCESS_KEY`.

Quick Commands:

```bash
echo 'example-passwd4minIO' > ~/.secrets/minio/root_password
ansible-playbook minio_containers.yml minio_vms.yml
ansible-playbook -i lab.inventory.proxmox.yml minio.yml
# can be destroyed with
ansible-playbook -i lab.inventory.proxmox.yml pve_remove_guests.yml -e host_list=minio,miniolb
```

# Playbooks

## `minio_containers.yml`

Creates the `nginx` LXC containers with the custom variables set directly in the playbook.

```bash
ansible-playbook minio_vms.yml
```

## `minio_vms.yml`

Creates the `minio` VMs with the custom variables set directly in the playbook.

```bash
ansible-playbook minio_vms.yml
```

## `minio.yml`

Contains two plays, one that configures `keepalived` and `nginx` on the LXC containers and one that configures `minio` on the VMs.

```bash
ansible-playbook -i lab.inventory.proxmox.yml minio.yml
```
