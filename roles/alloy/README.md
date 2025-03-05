Role `alloy`
============

A role to create a set of `*.alloy` files for use with `grafana.grafana.alloy` and `CONFIG_FILE` configured as a directory in which Grafana Alloy will find all the `*.alloy` configuration files.

Caution: Changes to the configuration files will cause a Restart of the `alloy` service which is _not_ installed by this role. It does not make sense to use this role without installing Alloy. Because the restart happens after all roles have been processed, this will not cause errors when the `grafana.grafana.alloy` role is called after this role.

Role Variables
--------------

**`alloy_configs_dir`**

Default: `/opt/alloy`

The directory to contain all the alloy files.

**`alloy_loki_journal_enabled`**

Default: `true`

Add the Loki Journal config file when true.

**`alloy_loki_var_logs_enabled`**

Default: `true`

Add the Loki `/var/logs` config file when true.

**`alloy_loki_uri`**

Default: `http://loki/loki/api/v1/push`

The URI for the `loki.write` Endpoint.

**`alloy_mimir_uri`**

Default: `http://mimir/api/v1/push`

The URI for the `prometheus.remote_write` Endpoint.

**`alloy_prometheus_minio_enabled`**

Default: `false`

Add the Prometheus Minio config file when true.
This is still a work in progress. Not sure anything of value is exported right now.

**`alloy_prometheus_pve_enabled`**

Default: `false`

Add the Prometheus PVE config file when true.
Try the these dashboards with this.

- https://grafana.com/grafana/dashboards/10347-proxmox-via-prometheus/
- https://grafana.com/grafana/dashboards/19484-proxmox-general/

**`alloy_prometheus_unix_enabled`**

Default: `true`

Add the Prometheus Unix config file when true.
Try the these dashboards with this.

- https://grafana.com/grafana/dashboards/14513-linux-exporter-node/

Example Playbook
----------------

```yml
---
- name: Configure Alloy on Generic hosts in the Lab network.
  hosts: study_vm # Changed to be the group or hosts desired

  # The strange quoting is so the resulting configuration file
  # will have the values quoted.
  vars:
    alloy_env_file_vars:
      CONFIG_FILE: '"/opt/alloy"'
      CUSTOM_ARGS: '"--server.http.listen-addr=0.0.0.0:12345"'
      RESTART_ON_UPGRADE: true

  roles:
    - role: alloy # local to this repository
    - role: grafana.grafana.alloy
```

```yml
---
- name: Configure Alloy on the Minio hosts in the Lab network.
  hosts: minio

  # The strange quoting is so the resulting configuration file
  # will have the values quoted.
  vars:
    alloy_env_file_vars:
      CONFIG_FILE: '"/opt/alloy"'
      CUSTOM_ARGS: '"--server.http.listen-addr=0.0.0.0:12345"'
      RESTART_ON_UPGRADE: true
    alloy_prometheus_minio_enabled: true

  roles:
    - role: alloy # local to this repository
    - role: grafana.grafana.alloy
```

```yml
---
- name: Configure Alloy on the Proxmox PVE hosts in the Lab network.
  hosts: proxmox_hosts

  # The strange quoting is so the resulting configuration file
  # will have the values quoted.
  vars:
    alloy_env_file_vars:
      CONFIG_FILE: '"/opt/alloy"'
      CUSTOM_ARGS: '"--server.http.listen-addr=0.0.0.0:12345"'
      RESTART_ON_UPGRADE: true
    alloy_prometheus_pve_enabled: true

  roles:
    - role: cloudcodger.proxmox_openssh.pve_exporter
    - role: alloy # local to this repository
    - role: grafana.grafana.alloy
```
