Setting up a local DNS server using **BIND on FreeBSD** for use with Kubernetes ingress and services involves several key steps:

### **1. Install BIND on FreeBSD**

```shell
pkg install bind916
```

Enable and start BIND:

```shell
sysrc named_enable="YES" service named start
```

---

### **2. Configure BIND for Kubernetes DNS Resolution**

Edit the main configuration file `/usr/local/etc/namedb/named.conf`:

```shell
options {
    directory "/usr/local/etc/namedb";
    listen-on port 53 { any; };
    allow-query { any; };
    recursion yes;
    forwarders { 8.8.8.8; 8.8.4.4; };
};

zone "myk8s.local" IN {
    type master;
    file "myk8s.local.db";
    allow-update { none; };
};
```

---

### **3. Create the Kubernetes DNS Zone File**

Create the file `/usr/local/etc/namedb/myk8s.local.db` with the following content:

```shell
$TTL 86400
@   IN  SOA ns.myk8s.local. root.myk8s.local. (
            2023121101 ; Serial
            3600       ; Refresh
            1800       ; Retry
            604800     ; Expire
            86400 )    ; Minimum TTL

; Define Nameserver
    IN  NS  ns.myk8s.local.

; Kubernetes Control Plane and Services
ns              IN  A   192.168.20.7
apiserver       IN  A   192.168.20.7

; Example Kubernetes Ingress
web-service     IN  A   192.168.20.10
api-service     IN  A   192.168.20.11
```

---

### **4. Update FreeBSD `resolv.conf`**

Set the local DNS server as the primary resolver:

```shell
echo "nameserver 192.168.20.7" > /etc/resolv.conf
```

---

### **5. Kubernetes Ingress and Service Configuration**

Configure Kubernetes ingress to use the custom domain `myk8s.local`.

**Example Kubernetes Ingress Resource:**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  namespace: default
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: web-service.myk8s.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web-app
            port:
              number: 80
```

---

### **6. Testing DNS and Services**

1. **Check BIND Service Status:**

    `service named status`
    
2. **Test DNS Resolution:**
    
    `nslookup web-service.myk8s.local 192.168.20.7`
    
3. **Verify Kubernetes Access:**
    
    `kubectl get ingress -A curl http://web-service.myk8s.local`
    

---

### Key Considerations:

- **Networking:** Ensure Kubernetes networking policies allow access from the local DNS server.
- **Firewall:** Allow port 53 on the FreeBSD server.
- **DNS Updates:** Automate DNS updates using Kubernetes external-dns if needed.

This setup ensures Kubernetes ingress and services are resolved through a local DNS server, enabling easier internal access in a network-restricted environment.Setting up a local DNS server using **BIND on FreeBSD** for use with Kubernetes ingress and services involves several key steps:
