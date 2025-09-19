# A Grafana Stack using Microservices

The Grafana setup contains a group of different services that all work together in order to provide a Grafana web site. There are two basic methods for the deployment of the Grafana Stack.

This conflicts with the [Kubernetes k0s](Kubernetes_k0s.md) showcase as they both use the same hostnames and IPs for the Kubernetes cluster. If you want to create both clusters at the same time, one of them will need to be changed.

1. A Monolithic approach
2. A Micro-services approach

This showcase covers the Micro-services approach.

As of this update, the [MinIO](MinIO.md) showcase no longer works. The software has changed owndership and can no longer be installed as described. For this reason, this showcase no longer uses MinIO, which has been replaced with [Ceph Rados Gateway](Ceph-RGW.md).

## Prerequisites

These are in addition to the Python and Ansible items mentioned in the base `README.md`.

- The `kubectl` utility installed on the Ansible control host.
- The `helm` utility installed on the Ansible control host.
- The Grafana Helm Repository added (`helm repo add grafana https://grafana.github.io/helm-charts`).
- the MetalLB Helm Repository added (`helm repo add metallb https://metallb.github.io/metallb`).
- The Helm repositories updated (`helm repo update`).
- [Ceph Rados Gateway](Ceph-RGW.md) installed in the Proxmox VE cluster.
- RadowGW `admin` user.
- RadosGW `admin` user credentials in:
  - ~/.secrets/radosgw/AWS_ACCESS_KEY_ID
  - ~/.secrets/radosgw/AWS_SECRET_ACCESS_KEY
- RadosGW buckets `loki`, `loki-admin`, `loki-ruler`, `mimir`, `mimir-alertmanager`, and `mimir-ruler`.

# Components

- [s3lb](#install-rgw-load-balancer) NGINX as a Ceph RGW Load Balancer.
- [Kubernetes cluster](#install-kubernetes) for all the microservices.
- [MetalLB](#install-metallb) for Kubernetes Load Balancer.
- [Rook](https://rook.io/) for Kubernetes persistent storage.
- [Grafana Mimir](https://grafana.com/oss/mimir/) for Metrics.
- [Grafana Loki](https://grafana.com/oss/loki/) for Logs.
- [Grafana](https://grafana.com/grafana/) for Visualization.
- [Grafana Alloy](https://grafana.com/docs/alloy/latest/) for log and metrics Collection.

## Install RGW Load Balancer

Create and configure the `s3lb` CTs with the [s3.yml](../s3.yml) playbook.

This requires the DNS be configured to resolve the `gadget1`, `gadget2` and `gadget3`. Otherwise, these names must be changed to IP addresses for the Proxmox VE cluster nodes.

Commands:

```bash
ansible-playbook -i lab s3.yml
```

## Install Kubernetes

Create a Kubernetes cluster using the [k0s_grafana.yml](../k0s_grafana.yml) playbook.
This creates a larger [`k0s` cluster](../lab/host_vars/local_k0s_grafana.yml) and adds network interfaces to each node.
 then the small one in the [Kubernetes k0s](Kubernetes_k0s.md) showcase and install `k0s` on them. This is like the [Kubernetes_k0s](docs/Kubernetes_k0s.md) showcase, except that it will add a second Hard Disk (for Rook `mon`s of 50G in size, as `/dev/sdb`) and a third Hard Disk (for Rook `OSD`s of 120G in size, as `/dev/sdc`) to the worker nodes.

Grafana Mimir, Grafana Loki and Grafana will all be installed inside this cluster. As well as, MetalLB and Rook.

Commands:

```bash
ansible-playbook -i lab k0s_grafana.yml

ssh ubuntu@ctrl1 'sudo k0s kubeconfig admin' > ~/.kube/config
# or
ssh -i ~/.ssh/cloudcodger ubuntu@ctrl1 'sudo k0s kubeconfig admin' > ~/.kube/config
# or
mkdir -p ~/.kube
ssh ubuntu@ctrl1 "sudo k0s kubeconfig create --groups system:masters showcase" > ~/.kube/config

# can be destroyed with
ansible-playbook -i lab pve_remove_guests.yml -e host_list=k0s
```

## Install MetalLB

The services need a load balancer in the k8s cluster and [MetalLB](https://metallb.io/) serves that requirement.

```bash
# kubectl create namespace metallb-system and install metallb via helm
helm upgrade -n metallb-system --install --create-namespace metallb metallb/metallb
# Wait for the pods to become ready
kubectl apply -f manifests/metallb-address-pool.yml

kubectl get IPAddressPool -n metallb-system
```

## Prepare k8s for External Rook

A specific release of the [Rook repository](https://github.com/rook/rook) was used because the most recent `release-1.18` produced errors that I did not have the time or energy to try and fix. The commands below use the `${REPO_PATH}/files` directory because `.gitignore` will prevent it from accidentally getting added to this repository.

If you want to make sure you are using the latest `release-1.17` files, you can follow these steps. Any files that are different can be copied over and tried. Note that `operator.yaml` must contain the line `ROOK_CSI_KUBELET_DIR_PATH: "/var/lib/k0s/kubelet"` because `k0s` uses that path. The `sed` command below can be used to update `operator.yaml`. Don't just copy the one from the `examples` directory or things will not work.

- Clone the repository `git clone git@github.com:rook/rook.git`.
- Check out `release-1.17`
- Copy over the needed scripts and manifests.
- Update `manifests/rook/operator.yaml` to include `ROOK_CSI_KUBELET_DIR_PATH: "/var/lib/k0s/kubelet"`!
- Run `rook-external`.

```bash
cd files
git clone git@github.com:rook/rook.git
cd rook
git checkout release-1.17
cd ../..
diff files/rook/deploy/examples/create-external-cluster-resources.py bin
diff files/rook/deploy/examples/import-external-cluster.sh bin
diff files/rook/deploy/examples/cluster-external.yaml manifests/rook
diff files/rook/deploy/examples/common-external.yaml manifests/rook
diff files/rook/deploy/examples/common.yaml manifests/rook
diff files/rook/deploy/examples/crds.yaml manifests/rook
diff files/rook/deploy/examples/operator.yaml manifests/rook # should have one line different
```

To update `operator.yaml`.

```bash
sed '/\# ROOK_CSI_KUBELET_DIR_PATH:/a\
  ROOK_CSI_KUBELET_DIR_PATH: "/var/lib/k0s/kubelet"
' manifests/rook/operator.yaml > manifests/rook/operator.yaml
```

Edit `bin/rook-external` and set `PVE_NODE` to the name or IP address to one of the PVE nodes.

Run `bin/rook-external`.

Patch the `cephfs` storage class to be the default class.

```bash
kubectl patch storageclass cephfs -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'   
```

## Install Rook

Install the `rook-ceph` application for the Persistent Volumes. This is a multi-step process.

1. Create CRDS, common resources and the Rook Ceph Operator Config ConfigMap.
2. Wait for the Rook Ceph Operator pod to become READY.
3. Create the CephCluster (external).
4. Wait for the cluster to be created and all pods to become READY.

```bash
kubectl apply -f manifests/rook/crds.yaml -f manifests/rook/common.yaml -f manifests/rook/common-external.yaml -f manifests/rook/operator.yaml
# wait for the `rook-ceph-operator-*` pod to become READY
kubectl apply -f manifests/rook/cluster-external.yaml
# wait for the cluster to have Phase=Ready Message="Cluster created successfully" and Health=HEALTH_OK
kubectl get cephcluster -n rook-ceph-external
```

## Create Secrets with the S3 login credentials

Create the `rgw-access-key-secret` secret in each of the two namespaces `mimr` and `loki`.

Assuming the RadosGW access key and secret are located in the [required](#prerequisites) location. The secrets can be created using the following commands.

```bash
kubectl create namespace loki
kubectl create secret generic rgw-access-key-secret -n loki --from-file=${HOME}/.secrets/radosgw
kubectl create namespace mimir
kubectl create secret generic rgw-access-key-secret -n mimir --from-file=${HOME}/.secrets/radosgw
```

## Install Grafana Loki

Install Grafana Loki using Helm. The `manifests/loki-custom-values.yml` file defines `loadBalancerIP: "192.168.6.176"` and may need to be changed for your setup.

```bash
helm upgrade -n loki --install --create-namespace loki grafana/loki -f manifests/loki-custom-values.yml
```

To check and see if Loki is working. Follow the instructions in the output from the `helm` command. Don't forget to use an IP addresses if `loki` does not resolve via DNS.

```bash
# Send data
curl -H "Content-Type: application/json" -XPOST -s "http://loki/loki/api/v1/push"  \
--data-raw "{\"streams\": [{\"stream\": {\"job\": \"test\"}, \"values\": [[\"$(date +%s)000000000\", \"fizzbuzz\"]]}]}"
# Query the data
curl "http://loki/loki/api/v1/query_range" --data-urlencode 'query={job="test"}' | jq .data.result
```

To uninstall Grafana Loki.

```bash
helm uninstall -n loki loki
```

## Install Grafana Mimir

Install Grafana Mimir using helm. The `manifests/mimir-custom-values.yml` file defines `loadBalancerIP: "192.168.6.175"` and may need to be changed for your setup.

```bash
helm upgrade -n mimir --install --create-namespace mimir grafana/mimir-distributed -f manifests/mimir-custom-values.yml
```

To check and see if Mimir is working. Run `curl http://mimir/prometheus/api/v1/labels` and make sure the results include `"status":"success"`. Don't forget to use an IP addresses if `mimir` does not resolve via DNS.

To uninstall Grafana Mimir.

```bash
helm uninstall -n mimir mimir
```

## Install Grafana

Install Grafana using helm. The `get secret` command below will pull the admin password that was generated. The `manifests/grafana-custom-values.yml` file defines `loadBalancerIP: "192.168.6.177"` and may need to be changed for your setup.

```bash
helm upgrade -n monitoring --install --create-namespace grafana grafana/grafana -f manifests/grafana-custom-values.yml

kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

The Grafana web site should be reachable at `http://grafana/` using the login `admin` and the password from the above `get secret`.

To uninstall Grafana.

```bash
helm uninstall -n monitoring grafana
```

### Loki Data Source

Login to Grafana and configure a Loki data source.

Use the endpoint of `http://loki-gateway.loki.svc.cluster.local/`.

If Grafana was running outside the cluster, the endpoint would be `http://loki/` (assuming you have DNS configured) or `http://192.168.6.176/` (or the IP address used for `loadBalancerIP` in `manifests/loki-custom-values.yml`).

### Mimir Data Source

Login to Grafana and configure a Prometheus data source (for mimir).

Use the endpoint of `http://mimir-nginx.mimir.svc/prometheus`.

If Grafana was running outside the cluster, the endpoint would be `http://mimir/prometheus` (assuming you have DNS configured) or `http://192.168.6.175/prometheus` (or the IP address used for `loadBalancerIP` in `manifests/mimir-custom-values.yml`).

## Install Grafana Alloy inside the cluster

Install Grafana Alloy so the cluster can be monitored.

```bash
helm install --namespace monitoring k8s-monitoring grafana/alloy -f manifests/k8s-monitoring-custom-values.yml
```

## Install Grafana Alloy on PVE nodes

Use the `pve_alloy.yml` playbook to install Alloy and `pve_exporter` on the PVE nodes.

```bash
ansible-playbook -i lab pve_alloy.yml
```
