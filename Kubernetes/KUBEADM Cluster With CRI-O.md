## LIGHTWEIGHT CONTAINER RUNTIME FOR KUBERNETES

##### Add the Kubernetes repository
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | 
	gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" |
    tee /etc/apt/sources.list.d/kubernetes.list
```

##### Add the CRI-O repository
```
curl -fsSL https://pkgs.k8s.io/addons:/cri-o:/prerelease:/main/deb/Release.key |
    gpg --dearmor -o /etc/apt/keyrings/cri-o-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/cri-o-apt-keyring.gpg] https://pkgs.k8s.io/addons:/cri-o:/prerelease:/main/deb/ /" |
    tee /etc/apt/sources.list.d/cri-o.list
```

##### Install the packages
```
apt update
apt install -y cri-o kubelet kubeadm kubectl
```

### Initializing your control-plane node[](https://k8s-docs.netlify.app/en/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#initializing-your-control-plane-node)

The control-plane node is the machine where the control plane components run, including etcd (the cluster database) and the API Server (which the kubectl command line tool communicates with).

1. (Recommended) If you have plans to upgrade this single control-plane `kubeadm` cluster to high availability you should specify the `--control-plane-endpoint` to set the shared endpoint for all control-plane nodes. Such an endpoint can be either a DNS name or an IP address of a load-balancer.
2. Choose a Pod network add-on, and verify whether it requires any arguments to be passed to `kubeadm init`. Depending on which third-party provider you choose, you might need to set the `--pod-network-cidr` to a provider-specific value.
3. (Optional) Since version 1.14, `kubeadm` tries to detect the container runtime on Linux by using a list of well known domain socket paths. To use different container runtime or if there are more than one installed on the provisioned node, specify the `--cri-socket` argument to `kubeadm init`. 
4. (Optional) Unless otherwise specified, `kubeadm` uses the network interface associated with the default gateway to set the advertise address for this particular control-plane node’s API server. To use a different network interface, specify the `--apiserver-advertise-address=<ip-address>` argument to `kubeadm init`. To deploy an IPv6 Kubernetes cluster using IPv6 addressing, you must specify an IPv6 address, for example `--apiserver-advertise-address=fd00::101`
5. (Optional) Run `kubeadm config images pull` prior to `kubeadm init` to verify connectivity to the gcr.io container image registry.

To initialize the control-plane node run:

```bash
kubeadm init <args>
```

To run `kubeadm init` again, you must first tear down the cluster

Runtime and Path to Unix domain socket
```
containerd => unix:///var/run/containerd/containerd.sock
CRI-O      => unix:///var/run/crio/crio.sock
Docker     => unix:///var/run/cri-dockerd.sock`
```

Remember to use your selected container socket
```bash
sudo kubeadm init --apiserver-advertise-address=192.168.100.150 --cri-socket=unix:///var/run/crio/crio.sock --pod-network-cidr=10.244.0.0/16
```

Success initialization
```
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

kubeadm join 192.168.100.150:6443 --token sgeduc.xgj392mgbj6jm6yq \
        --discovery-token-ca-cert-hash sha256:21e366ef474a1a02cff3b38a5322cc0d1fabb0e9fe4d2fe473962324a95948a8
```

Before Running Any Network  Addons
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Test Kubeadm Installation
```
kubeadm cluster-info
kubeadm get nodes
```

Autostart kubectl swap needs to be off the `swapoff -a` command cannot be used because is not persistent on boot. Meanwhile disabling the swap in `/etc/fstab` is not working well because systemd will auto mount swap partition if there's any. Thus using this command below is recommended: 

```
systemctl mask "dev-sdaXX.swap"
```

Reset Kubeadm Config
```
sudo kubeadm reset --cri-socket=unix:///var/run/crio/crio.sock
sudo rm -rf $HOME/.kube/config /etc/cni/net.d/
```

## Install Flanel
```
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

## Install Cillium
```
CILIUM_CLI_VERSION=$(curl -s https://raw.githubusercontent.com/cilium/cilium-cli/main/stable.txt)
CLI_ARCH=amd64
if [ "$(uname -m)" = "aarch64" ]; then CLI_ARCH=arm64; fi
curl -L --fail --remote-name-all https://github.com/cilium/cilium-cli/releases/download/${CILIUM_CLI_VERSION}/cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
sha256sum --check cilium-linux-${CLI_ARCH}.tar.gz.sha256sum
sudo tar xzvfC cilium-linux-${CLI_ARCH}.tar.gz /usr/local/bin
rm cilium-linux-${CLI_ARCH}.tar.gz{,.sha256sum}
```

### Remove the node

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

### Clean up the control plane

You can use `kubeadm reset` on the control plane host to trigger a best-effort clean up.