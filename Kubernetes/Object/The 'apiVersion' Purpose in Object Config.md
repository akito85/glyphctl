The standard format is:

```
<group>/<version>
```

**Here's a breakdown:**

- **Group:**
    - Represents the API group, which is a logical grouping of related resources.
    - Common groups include:
        - `v1`: Core API group for basic resources like Pods, Services, and Deployments.
        - `apps/v1`: API group for advanced applications like Deployments and StatefulSets.
        - `batch/v1`: API group for batch jobs.
        - `networking.k8s.io/v1`: API group for network resources like Ingresses and NetworkPolicies.
- **Version:**
    - Specifies the version of the API within the group.
    - Versions are typically numbered sequentially, with higher numbers indicating newer versions.

**Example:**

YAML

```
apiVersion: apps/v1
kind: Deployment
```

Use code [with caution.](/faq#coding)

**Why is the Standard Format Important?**

- **Compatibility:** Using the correct `apiVersion` ensures that your Kubernetes resources are compatible with the API server version.
- **Feature Support:** Different API versions may support different features. Using the appropriate version allows you to leverage the latest features.
- **Backward Compatibility:** Kubernetes strives to maintain backward compatibility, but it's always recommended to use the latest stable API versions to avoid potential issues.

**Additional Notes:**

- **API Versioning:** Kubernetes uses a versioning scheme to manage changes to the API. This involves introducing new API versions with additional features or modifications.
- **API Stability:** Kubernetes API versions can be stable or unstable. It's generally recommended to use stable API versions for production deployments to avoid breaking changes.
- **Custom Resource Definitions (CRDs):** For custom resources, you can define your own API groups and versions.

By adhering to the standard `apiVersion` format and understanding the underlying concepts, you can effectively define and manage your Kubernetes resources.