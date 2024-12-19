### **How to Create a kubeconfig Using a ServiceAccount Token**

Follow these steps to create a kubeconfig file using a Kubernetes ServiceAccount token:

---

### **Step 1: Create a ServiceAccount**

```yaml
apiVersion: v1 
kind: ServiceAccount 
metadata:   
	name: my-serviceaccount   
	namespace: default
```

Apply the configuration:

```bash
kubectl apply -f my-serviceaccount.yaml
```

---

### **Step 2: Create a Role and RoleBinding**

```yaml
apiVersion: rbac.authorization.k8s.io/v1 
kind: Role metadata:   
name: my-role   
namespace: default 
  rules: 
	- apiGroups: [""]   
	  resources: ["pods", "services", "endpoints"]   
	  verbs: ["get", "list", "watch"]
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: my-rolebinding
  namespace: default
subjects:
- kind: ServiceAccount
  name: my-serviceaccount
  namespace: default
roleRef:
  kind: Role
  name: my-role
  apiGroup: rbac.authorization.k8s.io
```

Apply the files:

bash

Copy code

`kubectl apply -f my-role.yaml -f my-rolebinding.yaml`

---

### **Step 3: Extract the ServiceAccount Token**

1. Find the ServiceAccount secret name:
    
    bash
    
    Copy code
    
    `kubectl get sa my-serviceaccount -o jsonpath="{.secrets[0].name}" -n default`
    
2. Extract the token:
    
    bash
    
    Copy code
    
    `kubectl get secret <SECRET_NAME> -o jsonpath="{.data.token}" -n default | base64 -d`
    

---

### **Step 4: Extract the Kubernetes API Server URL**

bash

Copy code

`kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}'`

---

### **Step 5: Extract the CA Certificate**

bash

Copy code

`kubectl get secret <SECRET_NAME> -o jsonpath="{.data['ca\.crt']}" -n default | base64 -d > ca.crt`

---

### **Step 6: Create the kubeconfig File**

Create a file called `kubeconfig.yaml`:

yaml

Copy code

`apiVersion: v1 kind: Config clusters: - name: my-cluster   cluster:     certificate-authority: ./ca.crt     server: https://<K8S_API_SERVER_URL> contexts: - name: my-context   context:     cluster: my-cluster     user: my-serviceaccount current-context: my-context users: - name: my-serviceaccount   user:     token: <SERVICEACCOUNT_TOKEN>`

---

### **Step 7: Use the kubeconfig File**

bash

Copy code

`kubectl --kubeconfig=kubeconfig.yaml get pods`

---

### **Summary:**

- **Best Practice:** Configure a kubeconfig file using a ServiceAccount token for external access.
- **Avoid Creating a Pod with `kubectl`:** Use an external kubeconfig to securely authenticate using the ServiceAccount token.
- **Access Kubernetes Services:** Use `NodePort`, `LoadBalancer`, or `Ingress` for external access. Use `ClusterIP` for internal services.