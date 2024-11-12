# The `custom_vars` directory

This directory does not get automatically parsed by Ansible like the `groups_vars` and `host_vars` directories. The files in this directory must be included using the `vars_files:` directive inside a playbook.

Because many of the showcase playbooks must be run with `hosts: localhost` and require different values for the same set of variables. This means they cannot all be set in `group_vars/all.yml` nor `host_vars/localhost.yml`.
Instead, they are set in files under this directory and get included using the `vars_files:`.

They could also have been set directly inside the playbook or in other ways. This method just shows how you can place variables in a nice clean YAML file outside of your playbook for this situation.
