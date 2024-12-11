Creating a local DNS server for Kubernetes services and ingress can simplify service discovery and routing within the cluster. Here’s a step-by-step guide on how to set up a DNS server using **CoreDNS** and configure Kubernetes services with it.

---

### **1. Deploy CoreDNS as Local DNS Server**

CoreDNS can serve as a DNS server within a Kubernetes cluster.

#### **Install CoreDNS on the Virtual Machine:**

1. Install CoreDNS:
    
```bash
wget https://github.com/coredns/coredns/releases/download/v1.10.1/coredns_1.10.1_linux_amd64.tgz tar -xvzf coredns_1.10.1_linux_amd64.tgz chmod +x coredns sudo mv coredns /usr/local/bin/
```
    
2. Create a CoreDNS configuration file `/etc/coredns/Corefile`:

```lua
.:53 {
    log
    errors
    health
    forward . 8.8.8.8
}

cluster.local:53 {
    log
    errors
    kubernetes cluster.local in-addr.arpa ip6.arpa {
        endpoint https://192.168.20.7:6443
        tls /path/to/cert /path/to/key /path/to/ca-cert
        pods insecure
        fallthrough in-addr.arpa ip6.arpa
        ttl 30
    }
}
```
    
3. Start CoreDNS:
    
```bash
sudo coredns -conf /etc/coredns/Corefile
```

---

### **2. Configure Kubernetes to Use CoreDNS**

1. Update the Kubernetes `kubelet` configuration to use the DNS server:
    
    - Edit the Kubernetes config file `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf`:
```bash
--cluster-dns=192.168.20.7 --cluster-domain=cluster.local
```

2. Restart `kubelet`:

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

---

### **3. Kubernetes Ingress Setup**

1. Deploy an ingress controller such as **NGINX** or **Traefik**:
    
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```
    
2. Create a test service and ingress:
    
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: default
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080

---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
  namespace: default
spec:
  rules:
    - host: myapp.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-service
                port:
                  number: 80
```
    
3. Add a DNS entry in the CoreDNS `Corefile`:
    
```lua
myapp.local:53 { 
	forward . 192.168.20.7 
}
```
 
4. Reload CoreDNS.

---

### **Testing the Setup**

- Add a local DNS entry for `myapp.local` in `/etc/hosts`:

```bash
192.168.20.7 myapp.local
```
    
- Access the service:
    
```bash
curl http://myapp.local
```

---

To use **CoreDNS** in Kubernetes with a local IP for services and ingress, follow these steps. We'll configure CoreDNS to resolve internal Kubernetes services and external domains properly.

---

## **1. CoreDNS in Kubernetes Overview**

CoreDNS runs as a Kubernetes **Deployment** and provides DNS resolution for:

- **Services** in Kubernetes.
- External domains.
- Custom domains for services exposed via ingress.

---

## **2. Verify CoreDNS Deployment**

Check if CoreDNS is installed in your Kubernetes cluster:

```bash
kubectl -n kube-system get deployments coredns
```

If CoreDNS is not installed, deploy it using:

```bash
kubectl apply -f https://storage.googleapis.com/kubernetes-release/release/v1.26.0/examples/kubeadm/coredns.yaml
```

---

## **3. Update CoreDNS ConfigMap**

To configure CoreDNS for local IP DNS resolution:

1. **Edit CoreDNS ConfigMap**:

```bash
kubectl -n kube-system edit configmap coredns
```
    
2. **Add Custom DNS Entries**: Modify the `Corefile` configuration to handle internal Kubernetes services and local IPs.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health {
            lameduck 5s
        }
        ready
        kubernetes cluster.local in-addr.arpa ip6.arpa {
            fallthrough
        }
        hosts {
            # Custom DNS records for ingress and services
            192.168.20.7 ingress.k8s.local
            192.168.20.7 service1.k8s.local
            fallthrough
        }
        forward . 8.8.8.8
        cache 30
        loop
        reload
        loadbalance
    }
```    
3. **Save and Apply Changes**.
    

---

## **4. Deploy Kubernetes Ingress and Services**

### Create an Example Ingress Resource:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: ingress.k8s.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
```

### Create a Kubernetes Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
  namespace: default
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP
```
---

## **5. Update DNS on Client Machines**

Ensure that client machines can resolve Kubernetes service names:

- **Linux Clients**: Update `/etc/resolv.conf`:
    
```bash
nameserver 192.168.20.7
```
    
---

## **6. Test the Setup**

1. **DNS Lookup Test**:
    
```bash
nslookup ingress.k8s.local 192.168.20.7
```
    
2. **Access the Service via Ingress**:

```bash
http://ingress.k8s.local
``` 

---

### **How This Works:**

1. CoreDNS resolves Kubernetes services based on its internal database.
2. Custom DNS entries in CoreDNS map ingress and services to local IPs.
3. Client machines point to the Kubernetes control plane's IP (`192.168.20.7`) for DNS resolution.