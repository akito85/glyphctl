### **Definitions**
- **Role**: Grants access to all resources and API groups.
- **RoleBinding**: Binds the Role to a ServiceAccount.
- **ServiceAccount**: A Kubernetes ServiceAccount for authentication.
- **Deployment**: A sample deployment that uses the ServiceAccount.

---

### **1. Role (Granting access to all resources)**

Single groups with all resources

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: example-role
  namespace: default
rules:
  - apiGroups: ["*"]  # All API groups
    resources: ["*"]  # All resources (pods, services, deployments, etc.)
    verbs: ["*"]      # All actions (get, list, watch, create, delete, etc.)
```

Multiple groups custom resources

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: example-role
  namespace: default
rules:
  - apiGroups: [""] # "" means core API group
    resources: ["pods", "services", "endpoints", "configmaps"]
    verbs: ["get", "list", "watch", "create", "update", "delete"]
  - apiGroups: ["apps"]
    resources: ["deployments", "replicasets"]
    verbs: ["get", "list", "create", "update", "delete"]
  - apiGroups: ["networking.k8s.io"]
    resources: ["ingresses"]
    verbs: ["get", "list", "create", "update", "delete"]

```

---

### **2. RoleBinding (Linking Role to ServiceAccount)**


```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: example-role-binding
  namespace: default
subjects:
  - kind: ServiceAccount
    name: example-service-account
    namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: example-role

```

---

### **3. ServiceAccount (Creating the account)**


```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: example-service-account
  namespace: default

```

---

### **4. Deployment (Using the ServiceAccount)**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-deployment
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: example
  template:
    metadata:
      labels:
        app: example
    spec:
      serviceAccountName: example-service-account  # ServiceAccount reference
      containers:
        - name: example-container
          image: nginx:latest
          ports:
            - containerPort: 80

```

---

### **How Service Accounts Work**

1. **Access Flow:**
    
    - A **ServiceAccount** is created in the namespace.
    - The **Role** defines permissions for Kubernetes resources.
    - The **RoleBinding** links the Role to the ServiceAccount.
    - The **Deployment** specifies the ServiceAccount it should use.
2. **Authorization Process:**
    
    - When a pod starts, Kubernetes automatically mounts a token from the ServiceAccount into the pod (`/var/run/secrets/kubernetes.io/serviceaccount/token`).
    - Kubernetes APIs authenticate the pod using this token.

### **Accessing Kubernetes Resources:**

- When a Pod runs with a ServiceAccount, Kubernetes automatically mounts a token (`/var/run/secrets/kubernetes.io/serviceaccount/token`) inside the container.
- This token allows the Pod to authenticate with the Kubernetes API.
- Use `kubectl` or Kubernetes SDKs (like client libraries) to access cluster resources.

### **ServiceAccount in Deployment / Service / Ingress:**

- **Deployment:** Defined using `serviceAccountName` in the `spec` field.
- **Service/Ingress:** Do not directly use ServiceAccounts. Access to these resources should be granted through the Role assigned to the ServiceAccount.

### **Container Image for ServiceAccount Access**

To access cluster resources, use the **Kubernetes CLI (kubectl)** image or other Kubernetes client SDK images. Example Kubernetes management container image:

```yaml
containers:
  - name: kubectl
    image: bitnami/kubectl:latest
    command:
      - "/bin/sh"
      - "-c"
      - "sleep infinity"
```

### **Access Example**

```bash
kubectl get pods --token=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
```

If you need to access Kubernetes API using the ServiceAccount inside the cluster, you can create a pod running a `kubectl` image:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubectl-pod
  namespace: default
spec:
  serviceAccountName: example-sa
  containers:
  - name: kubectl
    image: bitnami/kubectl:latest
    command: ["sleep", "3600"]

```

Accessing the Pod from Outside the Cluster

1. Expose with a service using port forwarding
```bash

kubectl port-forward pod/kubectl-pod 8080:80 -n default
```

```yaml

apiVersion: v1
kind: Service
metadata:
  name: kubectl-service
  namespace: default
spec:
  selector:
    app: kubectl-pod
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080

```

3. Expose with a service using NodePort
```
apiVersion: v1
kind: Service
metadata:
  name: kubectl-service
  namespace: default
spec:
  selector:
    app: kubectl-pod
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
    nodePort: 30000
  type: NodePort
```

4. Expose with a service using ClusterIP (Ingress, Proxy, or LoadBalancer needed)
```yaml
apiVersion: v1
kind: Service
metadata:
  name: kubectl-service
  namespace: default
spec:
  selector:
    app: kubectl-pod
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: ClusterIP
```

---

### **Defining ServiceAccount in Other Resources**

1. **Service:** No need to define a ServiceAccount in a Service. Services expose pods and don’t manage security.
    
2. **Ingress:** Ingress also doesn’t require a ServiceAccount directly since it interacts with Services and doesn’t run inside a pod.

---

### **Best Practice**

- **Avoid creating a dedicated `kubectl` pod** in production for ServiceAccount access.
- Instead, **create an external kubeconfig** with the ServiceAccount token:

1. Extract the ServiceAccount token:
   
```bash
kubectl get secret $(kubectl get sa my-serviceaccount -o jsonpath='{.secrets[0].name}') -o jsonpath='{.data.token}' | base64 -d
```
    
2. Use the token and the Kubernetes API server endpoint to create a kubeconfig file. This allows external access using `kubectl` on your machine.