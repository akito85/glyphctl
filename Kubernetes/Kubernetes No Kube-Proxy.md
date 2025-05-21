### 1. For existing installations with `kube-proxy` running as a DaemonSet, remove it by using the following commands below.

```bash
kubectl -n kube-system delete ds kube-proxy
# Delete the configmap as well to avoid kube-proxy being reinstalled during a 
Kubeadm upgrade # (works only for K8s 1.19 and newer)
kubectl -n kube-system delete cm kube-proxy
# Run on each node with root permissions:
iptables-save | grep -v KUBE | iptables-restore
```
---
### 2. Install Cilium via Helm
Setup Helm Repository
```bash
helm repo add cilium https://helm.cilium.io/
```

The Cilium agent needs to be made aware of this information with the following configuration:
```bash
API_SERVER_IP=<your_api_server_ip>
# Kubeadm default is 6443
API_SERVER_PORT=<your_api_server_port>
helm install cilium cilium/cilium --version 1.17.4 \
    --namespace kube-system \
    --set kubeProxyReplacement=true \
    --set k8sServiceHost=${API_SERVER_IP} \
    --set k8sServicePort=${API_SERVER_PORT}
```

Cilium will automatically mount cgroup v2 filesystem required to attach BPF cgroup programs by default at the path `/run/cilium/cgroupv2`. To do that, it needs to mount the host `/proc` inside an init container launched by the DaemonSet temporarily. If you need to disable the auto-mount, specify `--set cgroup.autoMount.enabled=false`, and set the host mount point where cgroup v2 filesystem is already mounted by using `--set cgroup.hostRoot`. For example, if not already mounted, you can mount cgroup v2 filesystem by running the below command on the host, and specify `--set cgroup.hostRoot=/sys/fs/cgroup`.

```bash
mount -t cgroup2 none /sys/fs/cgroup
```
---
### 3. Verify Cilium Installation
Verify that Cilium has come up correctly on all nodes and is ready to operate:
```bash

kubectl -n kube-system get pods -l k8s-app=cilium
```

In above Helm configuration, the `kubeProxyReplacement` has been set to `true` mode. This means that the Cilium agent will bail out in case the underlying Linux kernel support is missing.

By default, Helm sets `kubeProxyReplacement=false`, which only enables per-packet in-cluster load-balancing of ClusterIP services.
Cilium’s eBPF kube-proxy replacement is supported in direct routing as well as in tunneling mode.

---
### 4. Validate the Setup
```bash
# Validate that the Cilium agent is running in the desired mode
kubectl -n kube-system exec ds/cilium -- cilium-dbg status | grep KubeProxyReplacement

# User verbose for full details
kubectl -n kube-system exec ds/cilium -- cilium-dbg status --verbose

# Check service
kubectl -n kube-system exec ds/cilium -- cilium-dbg service list
```