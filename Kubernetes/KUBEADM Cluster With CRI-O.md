## Pre-requisite

### 1. Repository Setup

##### Add the Kubernetes repository
```bash
KUBE_V=v1.33
curl -fsSL https://pkgs.k8s.io/core:/stable:/$KUBE_V/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/$KUBE_V/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

##### Add the CRI-O repository
```bash
CRIO_V=v1.33
curl -fsSL https://download.opensuse.org/repositories/isv:/cri-o:/stable:/$CRIO_V/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/cri-o-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/cri-o-apt-keyring.gpg] https://download.opensuse.org/repositories/isv:/cri-o:/stable:/$CRIO_V/deb/ /" | tee /etc/apt/sources.list.d/cri-o.list
```

##### Install the packages
```bash
sudo apt update
sudo apt install -y cri-o kubelet kubeadm kubectl podman
```

##### Hold the packages to prevent updates
```bash
sudo apt-mark hold kubelet kubeadm kubectl
```

##### Start the crio service
```bash
sudo systemctl daemon-reload 
sudo systemctl enable crio --now 
sudo systemctl start crio.service
```


### 2. Environment Setup

#### Disable Swap
```bash
# Disable all devices marked as swap in /etc/fstab
sudo swapoff -a

# Put scheduler swapoff
(crontab -l 2>/dev/null; echo "@reboot /sbin/swapoff -a") | crontab - || true

# Comment /etc/fstab swap mounting point
sudo sed -e '/swap/ s/^#*/#/' -i /etc/fstab   

# Prevent systemd automounted swap
sudo systemctl --all --type swap
sudo systemctl mask dev-sdaX.swap
```

#### Load Modules
```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

#### Sysctl Entries
```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

### Initializing your control-plane node

The control-plane node is the machine where the control plane components run, including etcd (the cluster database) and the API Server (which the kubectl command line tool communicates with).

1. (Recommended) If you have plans to upgrade this single control-plane `kubeadm` cluster to high availability you should specify the `--control-plane-endpoint` to set the shared endpoint for all control-plane nodes. Such an endpoint can be either a DNS name or an IP address of a load-balancer.
2. (Recommended) To set the `--pod-network-cidr` to a provider-specific value.
3. (Optional) To use different container runtime or if there are more than one installed on the provisioned node, specify the `--cri-socket` argument to `kubeadm init`. 
4. (Optional) To use a different network interface, specify the `--apiserver-advertise-address=<ip-address>` argument to `kubeadm init`. To deploy an IPv6 Kubernetes cluster using IPv6 addressing, you must specify an IPv6 address, for example `--apiserver-advertise-address=fd00::101`
5. (Optional) Run `kubeadm config images pull` prior to `kubeadm init` to verify connectivity to the gcr.io container image registry.


Runtime and Path to Unix domain socket
```
containerd => unix:///var/run/containerd/containerd.sock
CRI-O      => unix:///var/run/crio/crio.sock
Docker     => unix:///var/run/cri-dockerd.sock`
```

Remember to use your selected container socket
```bash
# cilium
sudo kubeadm config images pull
sudo kubeadm init --apiserver-advertise-address=192.168.100.150 --cri-socket=unix:///var/run/crio/crio.sock --pod-network-cidr=10.1.1.0/24 
```

Success initialization
```bash
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.100.150:6443 --token gj0lmr.w70nq5s7wicrbbda \
        --discovery-token-ca-cert-hash sha256:026878ed29b623bd95dfe18b9ebedf15aeaf6308ded64dcb957e8b7a29c53c4e
```

Before Running Any Network  Addons (***admin.conf*** or ***kubelet.conf***)
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Test Kubeadm Installation
```bash
kubectl cluster-info
kubectl get nodes -o wide
```

To run `kubeadm init` again, you must first tear down the cluster

### 3. Install CNI
#### Flanel
Install and Apply Flannel to Kubectl
```
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

#### Cilium
Install Cilium CLI
```bash
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
CLI_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
```

Apply Cilium to Kubectl
```bash
cilium install
```

### 4. Node Management
#### Join the Node to a Cluster

Join Command
```bash
kubeadm join 192.168.100.150:6443 --token g9c56i.4d5uxztxb8qclbu9 --discovery-token-ca-cert-hash sha256:b6c1d8d4b68d30c0ece8189c51c05c8549d5c83d1218008ae3a188d6cc047bbd
```

If The Token Expired Reprint Join Command
```bash
kubeadm token create --print-join-command
```

#### Remove the Node from a Cluster

Talking to the control-plane node with the appropriate credentials, run:

```bash
kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
kubectl delete node <node name>
```

Then, on the node being removed, reset all `kubeadm` installed state:

```bash
kubeadm reset
```

The reset process does not reset or clean up iptables rules or IPVS tables. If you wish to reset iptables, you must do so manually:

```bash
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
```

If you want to reset the IPVS tables, you must run the following command:

```bash
ipvsadm -C
```

If you wish to start over simply run `kubeadm init` or `kubeadm join` with the appropriate arguments.

---
#### Node Labels
##### 1. How to read node labels in Kubernetes

You can list labels in this fashion:

```shell
kubectl get nodes --show-labels
```

If you want to know the details for a specific node, use this:

```bash
kubectl label --list nodes node_name
```

> [!NOTE]
> > The labels are in form of key-value pair. They must begin with a letter or number, and may contain letters, numbers, hyphens, dots, and underscores, up to 63 characters each.

##### 2. How to assign label to a node

Label Node (ROLE):
```bash
kubectl label node <node_name> node-role.kubernetes.io/worker=<new_role_name>
```

Label Node:
```bash
kubectl label node <node_name> <key>=<value>
```

Confirm the pod labelling:

```bash
kubectl label --list nodes <node_name> | grep -i <key>
```

I used the grep command to weed out unnecessary details and focus on the label.



### 5. Troubleshooting

#### Check kubelet logs

Check kubelet logs via journalctl
```bash
sudo journalctl -xeu kubelet
```

#### Check cilium logs

Get all cilium pods
```bash
kubectl -n kube-system get pods -l k8s-app=cilium
```

Check the logs of the most restarted cilium pods
```bash
kubectl -n kube-system  logs --timestamps cilium-xxx
```

Go for detailed logs
```bash
kubectl -n kube-system exec cilium-xxx -- cilium-dbg status
```

Use this script to debug cilium in all nodes and run `./k8s-cilium-exec.sh cilium-dbg status`
```bash
curl -sLO https://raw.githubusercontent.com/cilium/cilium/main/contrib/k8s/k8s-cilium-exec.sh
chmod +x ./k8s-cilium-exec.sh
```

---

#### Error logs
**ERROR**
```bash
Defaulted container "cilium-agent" out of: cilium-agent, config (init), mount-cgroup (init), apply-sysctl-overwrites (init), mount-bpf-fs (init), clean-cilium-state (init), install-cni-binaries (init)
Error from server (BadRequest): container "cilium-agent" in pod "cilium-6zk7d" is waiting to start: PodInitializing
```

AND

```bash
"Could not open resolv conf file." err="open /run/systemd/resolve/resolv.conf: no such file or directory"
```

SOLUTION

1. Edit kubelet config, change `resolvConf: <path-to-your-resolv.conf>`
```bash
kubectl edit cm -n kube-system kubelet-config
```

2. Download config from cluster
```bash
kubeadm upgrade node phase kubelet-config
```

3. Restart kubelet
```bash
systemctl restart kubelet
```

4. Recreate CoreDNS pods (restart rollout or delete existing pods)
```bash
kubectl delete pods 'CoreDNS'
```

5. Run the `rollout restart` command below to restart the pods one by one without impacting the deployment.
```bash
kubectl rollout restart deployment <deployment_name>
```

---

**ERROR**
```bash
Error in configuration:
* unable to read client-cert /var/lib/kubelet/pki/kubelet-client-current.pem for default-auth due to open /var/lib/kubelet/pki/kubelet-client-current.pem: permission denied
* unable to read client-key /var/lib/kubelet/pki/kubelet-client-current.pem for default-auth due to open /var/lib/kubelet/pki/kubelet-client-current.pem: permission denied
```

SOLUTION

1. Change folder permission
```bash
sudo chown $(id -u):$(id -g) /var/lib/kubelet/ -R
```

2. Restart kubelet
```bash
sudo systemctl kubelet restart
```

---

**ERROR**
```bash
Error: template: cilium/templates/cilium-envoy/servicemonitor.yaml:1:20: executing "cilium/templates/cilium-envoy/servicemonitor.yaml" at <include "envoyDaemonSetEnabled" .>: error calling include: template: cilium/templates/_helpers.tpl:153:11: executing "envoyDaemonSetEnabled" at <semverCompare ">=1.16" (default "1.16" .Values.upgradeCompatibility)>: error calling semverCompare: Invalid Semantic Version
```

SOLUTION

1. Try to disable hubble
```bash
cilium hubble disable
```

2. Restart kubelet
```bash
sudo systemctl kubelet restart
```

### Clean up the control plane

You can use `kubeadm reset` on the control plane host to trigger a best-effort clean up.

```bash
sudo rm -rf $HOME/.kube/config /etc/cni/net.d/
```