# Ceph Rados Gateway Configuration

Install Ceph and the RadosGW on the PVE cluster nodes.

Ceph requires dedicated disks on each PVE node for [OSD disks](#add-osd-disks). Any workarounds for this are not recommended and may cause issues with this showcase.

Ceph RGW (aka RadosGW), provides an AWS S3 API endpoint that is required for the [Grafana Microservices Stack](Grafana_Microservices_Stack.md).

## Pre-install

Add network interfaces for Ceph to use. One for Ceph Public and one for Ceph Cluster.

In this example, the PVE nodes each have 4 NICs, with `vmbr0` already configured on the first NIC.

- Public network interfaces as `vmbr1`
    - `gadget1`: `192.168.16.251/24`
    - `gadget2`: `192.168.16.252/24`
    - `gadget3`: `192.168.16.253/24`
- Cluster network interfaces as `vmbr2`
    - `gadget1`: `192.168.26.251/24`
    - `gadget2`: `192.168.26.252/24`
    - `gadget3`: `192.168.26.253/24`

These commands can be run on any of the PVE nodes to create them.

Best practice is to put these Ceph networks on different interfaces and even to make them 10 Gig. This is not required for a lab or Proof of Concept (PoC).

```bash
pvesh create /nodes/gadget1/network --iface=vmbr1 --autostart=yes --bridge_ports=enp2s0 --cidr=192.168.16.251/24 --comments="Ceph Public" --type=bridge
pvesh create /nodes/gadget1/network --iface=vmbr2 --autostart=yes --bridge_ports=enp3s0 --cidr=192.168.26.251/24 --comments="Ceph Cluster" --type=bridge
pvesh set /nodes/gadget1/network

pvesh create /nodes/gadget2/network --iface=vmbr1 --autostart=yes --bridge_ports=enp2s0 --cidr=192.168.16.252/24 --comments="Ceph Public" --type=bridge
pvesh create /nodes/gadget2/network --iface=vmbr2 --autostart=yes --bridge_ports=enp3s0 --cidr=192.168.26.252/24 --comments="Ceph Cluster" --type=bridge
pvesh set /nodes/gadget2/network

pvesh create /nodes/gadget3/network --iface=vmbr1 --autostart=yes --bridge_ports=enp2s0 --cidr=192.168.16.253/24 --comments="Ceph Public" --type=bridge
pvesh create /nodes/gadget3/network --iface=vmbr2 --autostart=yes --bridge_ports=enp3s0 --cidr=192.168.26.253/24 --comments="Ceph Cluster" --type=bridge
pvesh set /nodes/gadget3/network
```

If the PVE nodes only have one NIC, either change any references to the NICs and IPs to use the single interface, or (assuming the network switch supports VLAN tagging) use these commands to create VLAN based bridges. This VLAN setup was not tested but should work and allow the IP addresses in the showcase to work unchanged.

```bash
pvesh create /nodes/gadget1/network --iface=enp1s0.10 --type=vlan --autostart=yes
pvesh create /nodes/gadget1/network --iface=vmbr1 --autostart=yes --bridge_ports=enp1s0.10 --cidr=192.168.16.251/24 --comments="Ceph Public" --type=bridge
pvesh create /nodes/gadget1/network --iface=enp1s0.20 --type=vlan --autostart=yes
pvesh create /nodes/gadget1/network --iface=vmbr1 --autostart=yes --bridge_ports=enp1s0.20 --cidr=192.168.26.251/24 --comments="Ceph Cluster" --type=bridge

pvesh create /nodes/gadget2/network --iface=enp1s0.10 --type=vlan --autostart=yes
pvesh create /nodes/gadget2/network --iface=vmbr1 --autostart=yes --bridge_ports=enp1s0.10 --cidr=192.168.16.252/24 --comments="Ceph Public" --type=bridge
pvesh create /nodes/gadget2/network --iface=enp1s0.20 --type=vlan --autostart=yes
pvesh create /nodes/gadget2/network --iface=vmbr1 --autostart=yes --bridge_ports=enp1s0.20 --cidr=192.168.26.252/24 --comments="Ceph Cluster" --type=bridge

pvesh create /nodes/gadget3/network --iface=enp1s0.10 --type=vlan --autostart=yes
pvesh create /nodes/gadget3/network --iface=vmbr1 --autostart=yes --bridge_ports=enp1s0.10 --cidr=192.168.16.253/24 --comments="Ceph Public" --type=bridge
pvesh create /nodes/gadget3/network --iface=enp1s0.20 --type=vlan --autostart=yes
pvesh create /nodes/gadget3/network --iface=vmbr1 --autostart=yes --bridge_ports=enp1s0.20 --cidr=192.168.26.253/24 --comments="Ceph Cluster" --type=bridge
```

## Install the `ceph` software using the PVE commands.

- `squid` is version `19.2.2` at this time.
- The `init` sub-command includes the Public and Cluster networks.
    - `192.168.16.0/24` is the Public Network.
    - `192.168.26.0/24` is the Cluster Network.

```bash
# On the first node where ceph is installed
pveceph install --repository no-subscription --version squid
pveceph init --network 192.168.16.0/24 --cluster-network 192.168.26.0/24
pveceph mon create # This also creates the 'mgr' on the first node.
apt install radosgw

# On all remaining nodes where ceph is installed
pveceph install --repository no-subscription --version squid
pveceph mon create
pveceph mgr create
apt install radosgw
```

## Add OSD (disks)

On each node that has disks for ceph to use.

```bash
# Find available disks
lsblk
pveceph osd create /dev/sd[X]
```

## Add Ceph pools

To add a pool for CT and VM disks. Including the `--add_storages` will also add it as PVE storage for `Disk image, Container` content.
To add a pool for k8s.

```bash
pveceph pool create $POOL_NAME --add_storages
# for example
pveceph pool create ceph --add_storages 1
pveceph pool create grafana
```

## Add Ceph File System

Add a ceph file system for PVE storage for `Backup, ISO image, Container template` content and one for k8s.

The ceph file system requires a MetaData Server. Install one on each node to add redundancy.

```bash
# On each node, create an MDS for use by cephfs
pveceph mds create
# On one node, this also creates two pools for cephfs to use
pveceph fs create --name cephfs --pg_num 32 --add-storage 1
pveceph fs create --name grafanafs --pg_num 32
```

## Add RGW (Rados Gateway)

The commands:
- Create a directory for the key (`install` command).
- Create a key (or get an existing key) and place it in two files. The one under `/etc/pve/priv` is not part of a non-PVE configuration.
- Then `start` and `enable` the service.

```bash
# On each node
install -d -o ceph -g ceph /var/lib/ceph/radosgw/ceph-$(hostname -s)
ceph auth get-or-create client.$(hostname -s) mon 'allow rw' osd 'allow rwx' | tee /var/lib/ceph/radosgw/ceph-$(hostname -s)/keyring | tee /etc/pve/priv/ceph.client.$(hostname -s).keyring
systemctl start ceph-radosgw@$(hostname -s).service
systemctl enable ceph-radosgw@$(hostname -s).service
systemctl status ceph-radosgw@$(hostname -s).service
```

## Enable the Prometheus module

This module needs to be enabled for the Rook `create-external-cluster-resources.py` script.

```bash
ceph mgr module enable prometheus
```

## Create an S3 user

The user with `--admin` will be used for the [Grafana Microservices Stack](Grafana_Microservices_Stack.md). Configuring a normal user with the proper permissions is left as an exercise.

[Account](https://docs.ceph.com/en/latest/radosgw/account/)

```bash
radosgw-admin user create --uid=showcase --display-name="Showcase User"
radosgw-admin user create --uid=grafana --display-name="Grafana Stack Admin User" --admin
# List users
radosgw-admin user list
# Show a users info
radosgw-admin user info --uid showcase
radosgw-admin user info --uid grafana
```

## Save the auth values

Save the `aws_access_key_id` and `aws_secret_access_key` values from the output from creating the `grafana` user.

Using `vim` here so that the command history does not contain the values.

```bash
vim ~/.secrets/radosgw/AWS_ACCESS_KEY_ID
vim ~/.secrets/radosgw/AWS_SECRET_ACCESS_KEY
```

## Create S3 bucket

- Optional: Create the `s3lb` (the `s3.yml` playbook).
- Install the AWS CLI.
- Configure the CLI.
  - Save the `aws_access_key_id`, `aws_secret_access_key`, and `endpoint_url`.
- Create one or more buckets.

```bash
ansible-playbook -i lab s3.yml

brew install awscli
aws configure # get prompeted for the `aws_access_key_id` and `aws_secret_access_key`
aws configure set aws_access_key_id $(cat ~/.secrets/radosgw/AWS_ACCESS_KEY_ID)
aws configure set aws_secret_access_key $(cat ~/.secrets/radosgw/AWS_SECRET_ACCESS_KEY)
aws configure set endpoint_url http://192.168.6.251:7480/
# if using DNS and the 's3' domain name with the NGINX load balancer
aws configure set endpoint_url http://s3/
vim ~/.aws/config # add endpoint_url = http://192.168.6.251:7480/

aws s3api create-bucket --bucket example_bucket_name
# Example for the [Grafana Microservices Stack](Grafana_Microservices_Stack.md)
aws s3api create-bucket --bucket loki
aws s3api create-bucket --bucket loki-admin
aws s3api create-bucket --bucket loki-ruler
aws s3api create-bucket --bucket mimir
aws s3api create-bucket --bucket mimir-alertmanager
aws s3api create-bucket --bucket mimir-ruler
```

# Ceph Manager Dashboard

The Dashboard is not required. These commands can be used to configure it.

Commands that need to be run for `ceph-dashboard`.

```bash
# All PVE nodes
apt install ceph-mgr-dashboard
# On one PVE node
ceph mgr module ls
# To disable SSL
ceph config set mgr mgr/dashboard/ssl false
# or To create a self signed cert
ceph dashboard create-self-signed-cert
# or Set SSL certs on each PVE node
ceph dashboard set-ssl-certificate $(uname -n) -i /etc/pve/nodes/$(uname -n)/pve-ssl.pem
ceph dashboard set-ssl-certificate-key $(uname -n) -i /etc/pve/nodes/$(uname -n)/pve-ssl.key
# For Global SSL certs
# Assumes the files contain the cert and key.
ceph dashboard set-ssl-certificate -i /root/pve-ssl.pem
ceph dashboard set-ssl-certificate-key -i /root/pve-ssl.key
# Set the IP address per server
ceph config set mgr mgr/dashboard/gadget1/server_addr 192.168.6.251
ceph config set mgr mgr/dashboard/gadget2/server_addr 192.168.6.252
ceph config set mgr mgr/dashboard/gadget3/server_addr 192.168.6.253
# Create a user and password
vi ceph-admin-passwd # put a password in this file
ceph dashboard ac-user-create admin -i ceph-admin-passwd
ceph mgr module disable dashboard
ceph mgr module enable dashboard
```
