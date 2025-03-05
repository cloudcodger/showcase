# A Monolithic Grafana Stack

The Grafana setup contains a group of different services that all work together in order to provide a Grafana web site. There are two basic methods for the deployment of the Grafana Stack.

1. A Monolithic approach
2. A Micro-services approach

This showcase covers the Monolithic approach.

# Components

- A set of NGINX reverse proxy load balancers (See [Reverse Proxy](Reverse_Proxy.md))
- A backing store such as AWS S3 or [MinIO](https://min.io/) (See [MinIO](MinIO.md))
- [Grafana Mimir](https://grafana.com/oss/mimir/) for Metrics
- [Grafana Loki](https://grafana.com/oss/loki/) for Logs
- [Grafana](https://grafana.com/grafana/) for Visualization
- [Grafana Alloy](https://grafana.com/docs/alloy/latest/) for Collection

## NGINX Reverse Proxy servers

See the [Reverse Proxy](Reverse_Proxy.md) showcase. These are the load balancers for MinIO, Grafana Loki and Grafana Mimir. They are used in the configuration of later components, like the Alloy configurations. Creating them first will make each component able to use the load balancer when creating other components.

Quick Commands:

```bash
ansible-playbook rotary_containers.yml
ansible-playbook -i lab.inventory.proxmox.yml rotary.yml
```

## MinIO

See the [MinIO](MinIO.md) showcase. This will be the object store for [Grafana Mimir](https://grafana.com/oss/mimir/) and [Grafana Loki](https://grafana.com/oss/loki/) in order to avoid any costs that would be incurred by using an S3 bucket in AWS.

Comments have been added for what changes would need to be made in order to use AWS S3.

Steps:

1. Save `root` password
2. Create VMs (w/ second disks)
3. Configure VMs
4. Login to MinIO UI
    - Create API Key and Secret
    - Save key in `~/.secrets/minio/AWS_ACCESS_KEY_ID`
    - Save secret in `~/.secrets/minio/AWS_SECRET_ACCESS_KEY`.
    - Create Buckets `mimir` and `loki`.

Quick Commands:

```bash
echo 'example-passwd4minIO' > ~/.secrets/minio/root_password
ansible-playbook minio_vms.yml
ansible-playbook -i lab.inventory.proxmox.yml minio.yml
```

## Grafana Mimir and Grafana Loki

The `dogchow[123]` VMs are used for both Grafana Mimir and Grafana Loki. Both are installed in the same playbook.

If you have the NGINX Reverse Proxy set up, you should now be able to open http://rotary:9009/ in a browser and see a status page. Alternatively, http://dogchow1:8080/, http://dogchow2:8080/, and http://dogchow3:8080/ should provide the same status page for each of the systems.

Commands:

```bash
ansible-playbook dogchow_vms.yml
ansible-playbook -i lab.inventory.proxmox.yml dogchow.yml
```

## Grafana

Install Grafana in the `watchdog` LXC container.

Commands:

```bash
ansible-playbook watchdog_container.yml
ansible-playbook -i lab.inventory.proxmox.yml watchdog.yml
```
