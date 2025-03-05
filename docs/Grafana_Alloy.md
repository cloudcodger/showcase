# Monitoring with Grafana Alloy

A showcase of setting up and configuring Grafana Alloy to send data to Grafana Loki and Grafana Mimir.

The playbooks and configurations can be used with the [Grafana Microservices Stack](Grafana_Microservices_Stack.md), the [Grafana Momolithic Stack](Grafana_Monolithic_Stack.md), or your own Loki and Prometheus servers.

## Prerequisites

Grafana Loki and Grafana Mimir (or Prometheus) to use as endpoints.

# Install Alloy Configuration Files and Alloy itself

The `alloy` role is local to this repository because every individual deployment will desire different Alloy configuration files. Which is why this has not been placed in an Ansible Collection.

This role is specifically designed to be used with an Alloy Defaults file that contains `CONFIG_FILE: '"/opt/alloy"'` as seen in the `group_vars/all.yml` file. Note the settings for `alloy_loki_uri` and `alloy_mimir_uri` in that same file. These need to be configured for the correct endpoints for the two services.

The playbooks show here then call the `grafana.grafana.alloy` role which installs Grafana Alloy with the above defaults file.

Below are several playbooks that will 

Commands:

```bash
ansible-playbook -i lab.inventory.proxmox.yml build_alloy.yml
ansible-playbook -i lab.inventory.proxmox.yml dogchow_alloy.yml
ansible-playbook -i lab.inventory.proxmox.yml k0s_alloy.yml
ansible-playbook -i lab.inventory.proxmox.yml minio_alloy.yml
ansible-playbook -i lab.inventory.proxmox.yml pve_alloy.yml
```

# Installing Alloy on Proxmox VE hosts

This showcase also includes how to use `cloudcodger.proxmox_openssh.pve_exporter` to install the `prometheus-pve-exporter` Python module for collecting PVE metrics that can be scraped by Alloy. Look at `pve_alloy.yml` playbook and the `group_vars/proxmox_hosts.yml` file to see how to do it. The [Readme](https://github.com/cloudcodger/proxmox_openssh/blob/main/README.md) file contains more information.
