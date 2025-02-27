# Install and configure Proxmox VE

This first showcase will take multiple nodes and:

- Configure all hosts into a cluster named `lab`.
- Update the APT repositories to the "no-subscription" URLs.
- Add and Remove SSH authorized keys for the `root` user.
- Create a `images/00` directory under the `local` storage on each host and upload cloud-init image files there.
- Perform `apt dist-upgrade`, `reboot` and `apt autoremove` on the very first role execution.
- Change the `local` storage to allow `images` content.
- Create a `configs` storage for `snippets` content under Corosync file system.
  Caution: There is a size limitation here. This is NOT intended for large files or a large number of files.
- Create the Permissions Groups `Admin` and ACLs for the Administrator role.
- Create the Permissions User `s1m0ne@pve` and place it inside the `Admin` group.
- Create a User API token `s1m0ne@pve!ansible` and ACLs for the Administrator role.
- Create a local secrets directory `~/.pve_tokens`.
- Place the newly created API token in `~/.pve_tokens/lab-s1m0ne-pve-ansible.token`.

## Steps

1) Manually install [Proxmox Virtual Environment](https://www.proxmox.com/en/proxmox-virtual-environment/overview) onto the three computers.
2) Make sure the `pve_hosts.ini` file is correct for the names and IP addresses chosen.
3) Download any image files you want uploaded to each Proxmox VE node and place them in the `files` directory.
4) Manually configure `root` access (see below).

# Notices

The `group_vars/proxmox_hosts.yml` sets `ansible_python_interpreter: /usr/bin/python3`, which may need to be changed if the Proxmox hosts have a different python version installed in the future.

In order to test different storage types, these must either be created and correctly named during the initial installation, or after the initial setup, as is the case for `Ceph`.

# Proxmox VE `root` Access

After the inial installation of Proxmox VE, you must first configure passwordless SSH access to the computers as the `root` user. The following commands are just an example of how this can be done for SSH keys already created on your computer. The `~/.ssh/showcase` keypair is created by `bin/vminit` if it doesn't already exist.

If the Proxmox VE nodes had already been clustered together and a new VM was created to be the Ansible control host, `~/.ssh/authorized_keys` will now be a sym-link to `/etc/pve/priv/authorized_keys` that is shared across the cluster nodes. For example, when using `multipass launch` after having lost or destroyed the previous VM, you only need to run `ssh-copy-id` on one of the nodes and it will get synced over to the others. Or, if you stored the previous public/private keys, that are already configured in the `authorized_keys` file, you can simply put them back into the `./ssh` directory.

```bash
ssh-keygen -t ed25519 -N "" -f ~/.ssh/showcase # done by vminit
ssh-copy-id -i ~/.ssh/showcase.pub root@192.168.6.251
ssh-copy-id -i ~/.ssh/showcase.pub root@192.168.6.252
ssh-copy-id -i ~/.ssh/showcase.pub root@192.168.6.253
# If DNS is configured
ssh-copy-id -i ~/.ssh/showcase.pub root@gadget1
ssh-copy-id -i ~/.ssh/showcase.pub root@gadget2
ssh-copy-id -i ~/.ssh/showcase.pub root@gadget3
```

# Ansible roles

The `pve` Ansible playbook requires multiple collections and roles. Install them using the `requirements.yml` file.

```bash
ansible-galaxy collection install -r requirements.yml
```

# Configure the Proxmox cluster

You can now execute the playbook to configure the Proxmox VE nodes as described above.

```bash
ansible-playbook -i pve_hosts.ini pve.yml
```
