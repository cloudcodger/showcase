# Installing and Configuring a Kubernetes Cluster using 'k0s'

This showcase demonstrates how to create a Kubernetes cluster using the [k0s](https://k0sproject.io/). It uses the `cloudcodger.proxmox_client.cloud_init` role to create the VMs and `cloudcodger.k8s.k0s` role to configure the cluster.

CAUTION! There is an unknown and unresolved issue with a bare bones `k0s` cluster using this playbook right now. After the cluster has been up and running for some time (usually less than 24 hours), one or two of the control nodes will lock up and not be accessible. On the locked up nodes, one of the CPU cores will be at 100% utilization. Usually indicating that a process has entered an infinate loop. This will require a "hard stop" of the VM and then start it again. It seems like there is a bug somewhere in `k0s` that has not been found or fixed.

The `ssh` command below, assumes you have the `cloudcodger` public key in the `host_vars/localhost.yml` in `cloud_init_sshkeys:`, as can be seen on line 16. Because these are public keys, they are safe to place in this file as only the person with the matching private key would be able to login.

Quick commands:

```bash
ansible-playbook k0s_vm.yml

ansible-playbook -i lab.inventory.proxmox.yml k0s.yml

ssh -i ~/.ssh/cloudcodger ubuntu@ctrl1 'sudo cat /var/lib/k0s/pki/admin.conf' > ~/work/tmp/ctrl1.conf

ansible-playbook -i lab.inventory.proxmox.yml k0s_destroy.yml
```

One of the features of the `cloudcodger.proxmox_client.cloud_init` role is that it was writting with the ability to create multiple VMs to be used for installing Kubernetes. Make sure and read the [README.md](https://github.com/cloudcodger/proxmox_client/blob/main/roles/cloud_init/README.md). This showcase uses the default values in the role for Control nodes (the `ctrl` prefix), the first of which is also tagged as the `genesis` node, and worker nodes (the `work` prefix). When using this role, keep in mind that the IP addresses of the worker nodes follow directly after the IP addresses of the control nodes. This makes it impossible to change the number of control nodes after the original run without messing up your cluster. Select that number carefully so that you will not need to change it later.

The `cloudcodger.k8s.k0s` role then uses the PVE tags to configure a set of Control nodes and Worker nodes. Make sure and read the [README.md](https://github.com/cloudcodger/k8s/blob/main/roles/k0s/README.md) for the details.

# Playbooks

## `k0s_vm.yml`

Creates the QEMU VMs for `k0s`, using `custom_vars/k0s.yml` (See, [custom_vars](Custom_vars.md)).

```bash
ansible-playbook k0s_vm.yml
```

## `k0s.yml`

Configures the QEMU VMs as a Kubernetes cluster.

```bash
ansible-playbook -i lab.inventory.proxmox.yml k0s.yml
```

## `k0s_destroy.yml`

Destroys the `k0s` container and cleans up SSH keys.

```bash
ansible-playbook -i lab.inventory.proxmox.yml k0s_destroy.yml
```