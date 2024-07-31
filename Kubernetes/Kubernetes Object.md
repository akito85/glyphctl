## Background
_Kubernetes objects_ are persistent entities in the Kubernetes system. Kubernetes uses these entities to represent the state of your cluster. Specifically, they can describe:
- What containerized applications are running (and on which nodes)
- The resources available to those applications
- The policies around how those applications behave, such as restart policies, upgrades, and fault-tolerance

A Kubernetes object is a "record of intent"--once you create the object, the Kubernetes system will constantly work to ensure that object exists. By creating an object, you're effectively telling the Kubernetes system what you want your cluster's workload to look like; this is your cluster's _desired state_.

## Object Management
There are many methods to manage object in kubernetes as follows:

|Management technique|Operates on|Recommended environment|Supported writers|Learning curve|
|---|---|---|---|---|
|Imperative commands|Live objects|Development projects|1+|Lowest|
|Imperative object configuration|Individual files|Production projects|1|Moderate|
|Declarative object configuration|Directories of files|Production projects|1+|Highest|

### Imperative Commands
When using imperative commands, a user operates directly on live objects in a cluster. The user provides operations to the `kubectl` command as arguments or flags.

This is the recommended way to get started or to run a one-off task in a cluster. Because this technique operates directly on live objects, **it provides no history of previous configurations.**

Example:
```
kubectl create      deployment     nginx    --image=nginx
kubectl <command>   <subcomment>   <name>   <options>    
```

### Imperative object configuration
In imperative object configuration, the kubectl command specifies the operation (create, replace, update, delete etc.), optional flags and at least one file name. The file specified must contain a full definition of the object in YAML or JSON format.

Example:
```
kubectl create     -f              nginx.yaml
kubectl <command>  <file options>  <file location>
```

### Declarative object configuration

When using declarative object configuration, a user operates on object configuration files stored locally, however the user does not define the operations to be taken on the files. Create, update, and delete operations are automatically detected per-object by `kubectl`. This enables working on directories, where different operations might be needed for different objects.

---

## Object Spec and Status
The object `spec`, you have to set this when you create the object, providing a description of the characteristics you want the resource to have: its _desired state_.

The object `status` describes the _current state_ of the object, supplied and updated by the Kubernetes system and its components.

For example: in Kubernetes, a Deployment is an object that can represent an application running on your cluster. When you create the Deployment, you might set the Deployment `spec` to specify that you want three replicas of the application to be running. The Kubernetes system reads the Deployment spec and starts three instances of your desired application--updating the status to match your spec. If any of those instances should fail (a status change), the Kubernetes system responds to the difference between spec and status by making a correction--in this case, starting a replacement instance.

---

## Object Description
The object in kubernetes is described using `yaml` or `json` manifest file. Inside those files must a `spec`. For the required files inside the manifest file:
- `apiVersion` - Which version of the Kubernetes API you're using to create this object
- `kind` - What kind of object you want to create
- `metadata` - Data that helps uniquely identify the object, including a `name` string, `UID`, and optional `namespace`
- `spec` - What state you desire for the object



### Examples
Create Namespace
```
kubectl create namespace
```

Or apply using -n option to assign namespace
```yaml
kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml -n my-namespace
```

Create Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: myapp
  labels:
    app: myapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: myapp
          image: alpine:latest
          command: ['sh', '-c', 'while true; do echo "logging" >> /opt/logs.txt; sleep 1; done']
		  ports:
			- containerPort: 80
          volumeMounts:
            - name: data
              mountPath: /opt
      initContainers:
        - name: logshipper
          image: alpine:latest
          restartPolicy: Always
          command: ['sh', '-c', 'tail -F /opt/logs.txt']
          volumeMounts:
            - name: data
              mountPath: /opt
      volumes:
        - name: data
          emptyDir: {}
```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: eks-sample-linux-deployment
  namespace: eks-sample-app
  labels:
    app: eks-sample-linux-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: eks-sample-linux-app
  template:
    metadata:
      labels:
        app: eks-sample-linux-app
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/arch
                operator: In
                values:
                - amd64
      containers:
      - name: nginx
        image: docker.io/library/nginx
        ports:
        - name: http
          containerPort: 80
        imagePullPolicy: IfNotPresent
      nodeSelector:
        kubernetes.io/os: linux
```

Create Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: eks-sample-linux-service
  namespace: eks-sample-app
  labels:
    app: eks-sample-linux-app
spec:
  selector:
    app: eks-sample-linux-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```
