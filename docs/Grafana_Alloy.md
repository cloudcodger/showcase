# Monitoring with Grafana Alloy

A showcase of setting up and configuring Grafana Alloy to send data to Grafana Loki and/or Grafana Mimir (or Prometheus). This is not an extensive showcase for all the other showcase items. It has been limited to showing how to monitor the PVE cluster nodes.

The endpoints used can be provided from the [Grafana Microservices Stack](Grafana_Microservices_Stack.md), the [Grafana Momolithic Stack](Grafana_Monolithic_Stack.md), or your own Loki and Mimir (or Prometheus) servers. Both of the endpoints do not need to exist or be used. When collecting only metrics, only a Mimir (or Prometheus) enpoint is needed.

## Prerequisites

Grafana Loki and/or Grafana Mimir (or Prometheus) endpoints.

For example:

- LOKI: `http://loki/loki/api/v1/push`
- MIMIR: `http://mimir/api/v1/push`

# Alloy configuration

Alloy is installed using the `grafana.grafana.alloy` role. On the Proxmox VE nodes, the `cloudcodger.proxmox_openssh.pve_exporter` is also used to install the `pve_exporter` application that provides PVE cluster data.

The `lab/group_vars/all.yml` file set's two variables used by the `alloy` role for configuration.

- `alloy_env_file_vars`
    - set `--server.http.listen-addr` for connecting to alloy. This may not be desired in a production environment.
    - set `LOKI_URL` variable to the Loki endpoint for use inside other files.
    - set `MIMIR_URL` variable to the Mimir endpoint for use inside other files.
- `alloy_config`
    - Multi-line string that uses [`import.http`](https://grafana.com/docs/alloy/latest/reference/config-blocks/import.http/) to pull files from the [`garage`](Garage.md) server and reference the [declared](https://grafana.com/docs/alloy/latest/reference/config-blocks/declare/) configuration block from them.

Commands:

```bash
ansible-playbook -i lab pve_alloy.yml
```
