# A Grafana Stack using Microservices

The Grafana setup contains a group of different services that all work together in order to provide a Grafana web site. There are two basic methods for the deployment of the Grafana Stack.

1. A Monolithic approach
2. A Micro-services approach

This showcase covers the Micro-services approach.

## Prerequisites

These are in addition to the Python and Ansible items mentioned in the base `README.md`.

- The `kubectl` utility installed on the Ansible control host.
- The `helm` utility installed on the Ansible control host.
- The Grafana Helm Repository added (`helm repo add grafana https://grafana.github.io/helm-charts`).
- the MetalLB Helm Repository added (`helm repo add metallb https://metallb.github.io/metallb`).
- The Helm repositories updated (`helm repo update`).

# Components

- An Object Store such as AWS S3 or [MinIO](https://min.io/) (See [MinIO](MinIO.md))
- A Kubernetes cluster for all the microservices
- [MetalLB](https://metallb.io/) for Kubernetes Load Balancer
- [Rook](https://rook.io/) for Kubernetes persistent storage
- [Grafana Mimir](https://grafana.com/oss/mimir/) for Metrics
- [Grafana Loki](https://grafana.com/oss/loki/) for Logs
- [Grafana](https://grafana.com/grafana/) for Visualization
- [Grafana Alloy](https://grafana.com/docs/alloy/latest/) for Collection

## MinIO

See the [MinIO](MinIO.md) showcase. This will be the object store for [Grafana Mimir](https://grafana.com/oss/mimir/) and [Grafana Loki](https://grafana.com/oss/loki/) in order to avoid any costs that would be incurred by using an S3 bucket in AWS.

Comments have been added for what changes would need to be made in order to use AWS S3.

1. Save `root` password
2. Create VMs (w/ second disks)
3. Configure VMs
4. Login to MinIO UI
    - Create API Key and Secret
    - Save key in `~/.secrets/minio/AWS_ACCESS_KEY_ID`
    - Save secret in `~/.secrets/minio/AWS_SECRET_ACCESS_KEY`.
    - Create Buckets `loki`, `loki-admin`, `loki-ruler`, `mimir`, `mimir-alertmgr`, and `mimir-ruler`.

Quick Commands:

```bash
echo 'example-passwd4minIO' > ~/.secrets/minio/root_password
ansible-playbook minio_containers.yml
ansible-playbook minio_vms.yml
ansible-playbook -i lab.inventory.proxmox.yml minio.yml
```

## Install Kubernetes

Create a Kubernetes cluster using `k0s` that is larger the the small one in the [Kubernetes k0s](Kubernetes_k0s.md) showcase and install `k0s` on them. This is like the [Kubernetes_k0s](docs/Kubernetes_k0s.md) showcase, except that it will add a second Hard Disk (12G in size, as `/dev/sdb` and used for Rook `mon`s) and a third Hard Disk of (120G in size, as `/dev/sdc` and used for Rook `OSD`s) to the worker nodes.

Grafana Mimir, Grafana Loki and Grafana will all be installed inside this cluster. As well as, MetalLB and Rook.

Commands:

```bash
ansible-playbook k0s_grafana_vms.yml
ansible-playbook -i lab.inventory.proxmox.yml k0s.yml
ssh ubuntu@ctrl1 'sudo k0s kubeconfig admin' > ~/.kube/config
ssh -i ~/.ssh/cloudcodger ubuntu@ctrl1 'sudo k0s kubeconfig admin' > ~/.kube/config
or
mkdir -p ~/.kube
ssh ubuntu@ctrl1 "sudo k0s kubeconfig create --groups system:masters showcase" > ~/.kube/config
```

## Change worker nodes CPU

For each worker node. Change the CPU type to `host`.

## Install MetalLB

The services need a load balancer and `metallb` is the choice for the showcase.

```bash
# kubectl create namespace metallb-system
helm upgrade -n metallb-system --install --create-namespace metallb metallb/metallb
kubectl apply -f manifests/metallb-address-pool.yml

kubectl get IPAddressPool -n metallb-system
```

## Install Rook

Install the `rook-ceph` application for the Persistent Volumes. This is a multi-step process.

1. Create CRDS, common resources and the Rook Ceph Operator Config ConfigMap.
2. Wait for the Rook Ceph Operator pod to become READY
3. Create the StorageClass, PersistentVolumes and CephCluster on local storage
4. Wait for the cluster to be created and ready
5. Create a Ceph Filesystem
6. Wait for the FS to be ready
7. Create a Ceph StorageClass and mark it as `default`

```bash
kubectl apply -f manifests/rook-crds.yml -f manifests/rook-common.yml -f manifests/rook-operator.yml
# wait for the `rook-ceph-operator-*` pod to become READY
kubectl apply -f manifests/rook-cluster.yml
# wait for the cluster to have Phase=Ready Message="Cluster created successfully" and Health=HEALTH_OK
kubectl get cephcluster -n rook-ceph

kubectl apply -f manifests/rook-filesystem.yml
kubectl get cephfilesystem -n rook-ceph

kubectl apply -f manifests/rook-storageclass.yml
```

## Create Secrets with the S3 login credentials

Create a secret for the Access Key and Secret twice. Once in each of the two namespaces `mimr` and `loki`.

Assuming you created a MinIO access key, as described above and placed the values in the files `~/.secrets/minio.access.key` and `~/.secrets/minio.secret.key`. The secrets can be created using the following commands.

```bash
kubectl create namespace loki
kubectl create secret generic minio-access-key-secret -n loki --from-file=${HOME}/.secrets/minio
kubectl create namespace mimir
kubectl create secret generic minio-access-key-secret -n mimir --from-file=${HOME}/.secrets/minio
```

## Install Grafana Loki

Install Grafana Loki using Helm.

```bash
# kubectl create namespace loki
helm upgrade -n loki --install --create-namespace loki grafana/loki -f manifests/loki-custom-values.yml
```

To uninstall Grafana Loki.

```bash
helm uninstall -n loki loki
```

## Install Grafana Mimir

Install stall Grafana Mimir using helm.

```bash
# kubectl create namespace mimir
helm upgrade -n mimir --install --create-namespace mimir grafana/mimir-distributed -f manifests/mimir-custom-values.yml
```

Check to see if [Mimir](http://mimir/prometheus/api/v1/labels) is working.

To uninstall Grafana Mimir.

```bash
helm uninstall -n mimir mimir
```

## Install Grafana

Install Grafana. The `get secret` command below will pull the admin password that was generated.

```bash
helm upgrade -n monitoring --install --create-namespace grafana grafana/grafana -f manifests/grafana-custom-values.yml

kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

### Loki Data Source

Login to Grafana and configure the Loki data source.

Use the endpoint of `http://loki-gateway.loki.svc.cluster.local/`.

If Grafana was running outside the cluster, the endpoint would be `http://loki/` (assuming you have DNS configured) or `http://192.168.6.176/` (or the IP address used for `loadBalancerIP` in `manifests/loki/custom_values.yaml`).

To uninstall Grafana.

```bash
helm uninstall -n monitoring grafana
```

### Mimir Data Source

Login to Grafana and configure the Mimir data source. It gets added as a Prometheus data source.

Use the endpoint of `http://mimir-nginx.mimir.svc/prometheus`.

If Grafana was running outside the cluster, the endpoint would be `http://mimir/prometheus` (assuming you have DNS configured) or `http://192.168.6.175/prometheus` (or the IP address used for `loadBalancerIP` in `manifests/mimir/custom_values.yaml`).

## Install Grafana Alloy in the cluster

Install Grafana for visualization.

```bash
helm install --namespace monitoring k8s-monitoring grafana/alloy -f manifests/k8s-monitoring-custom-values.yml
```
