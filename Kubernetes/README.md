#  📘 Kubernetes Notes #

## Introduction to Kubernetes
Kubernetes (often abbreviated as K8s) is an open-source container orchestration platform designed to automate the deployment, scaling, and management of containerized applications.
Initially developed by Google and now maintained by the Cloud Native Computing Foundation (CNCF), Kubernetes has become the industry standard for running containerized workloads at scale.

## Key Features:

- **Automated Scheduling:** Places containers on optimal nodes.
- **Self-Healing:** Restarts failed containers and replaces them automatically.
- **Horizontal Scaling:** Increases or decreases the number of replicas based on load.
- **Service Discovery & Load Balancing:** Exposes containers to the network and balances traffic.
- **Storage Orchestration:** Automatically mounts storage systems.

## Example:
Imagine you have a web application running in 10 containers. Instead of manually starting/stopping them, Kubernetes will:
- Deploy them evenly across multiple servers.
- Replace any crashed containers automatically.
- Route traffic to available containers without downtime.

## Why We Need an Orchestration Tool
When working with a small number of containers, manual management may seem simple. But in production environments with hundreds or thousands of containers, manual management becomes inefficient and error-prone.
Challenges Without Orchestration:

- **Scaling**: Manually starting/stopping containers based on demand.
- **Load Balancing**: Distributing traffic manually.
- **High Availability:** Restarting failed containers quickly.
- **Networking:** Connecting containers securely and consistently.
- **Updates:** Deploying new versions without downtime.

Example Problem:
If you run 100 containers across 5 servers and one server fails, you must:
- Detect the failure.
- Start replacement containers on another server.
- Update your load balancer.
An orchestration tool like Kubernetes does all of this automatically.

## Why Kubernetes?
Kubernetes has emerged as the preferred orchestration tool for several reasons:
1. **Open Source**: Vendor-neutral and community-driven.
2. **Scalability**: Designed to handle large-scale workloads.
3. **Portability**: Works across on-premises, cloud, and hybrid environments.
4. **Extensibility**: Highly customizable through APIs and plugins.
5. **Resilience**: Automatic healing of failed containers and rescheduling.
6. **Comprehensive Ecosystem**: Supported by a wide range of tools and platforms.

## Architecture of Kubernetes
Kubernetes follows a master-worker architecture.
### Control Plane (Master Components)
The control plane manages the cluster’s state.
- #### API Server (kube-apiserver):
Entry point for all cluster commands and communications.
Example: kubectl get pods talks to the API Server.
- #### Controller Manager (kube-controller-manager):
Watches the cluster’s state and ensures it matches the desired state.
Example: If a pod crashes, it tells the scheduler to start a new one.
- #### Scheduler (kube-scheduler):
Decides which worker node will run a new pod based on resources, policies, etc.
- #### etcd:
Key-value store holding the cluster's state and configuration.

###  Worker Node Components
- #### Kubelet:
Runs on every worker node; ensures pods are running correctly.
- #### Kube-Proxy:
Handles network rules and routing for pods.
- #### Container Runtime:
Runs containers (e.g., Docker, containerd, CRI-O).

## **Kubernetes Architecture in a Nutshell**

- **Master Node**: The "manager" that makes decisions and keeps the cluster running.
- **Worker Node**: The "worker" that runs your applications.
- **Pods**: The "workers" of your apps, running one or more containers.
- **Additional Components**: Helpers like ConfigMaps, Secrets, Ingress, and Namespaces that make managing apps easier.

---

## **Example Workflow**

1. You tell Kubernetes to run an app using `kubectl`.
2. The **API Server** receives your request and forwards it to the **Scheduler**.
3. The **Scheduler** assigns the app to a **Worker Node**.
4. The **Kubelet** on the Worker Node starts the app in a **Pod**.
5. The **Controller Manager** ensures the app stays running, and **etcd** stores all the details.
6. If the app needs to talk to other apps, **Kube-proxy** handles the networking.

---


## Lifecycle of a Pod
A Pod is the smallest deployable unit in Kubernetes.
Pod Lifecycle Phases:
- **Pending** – Pod is accepted but not yet scheduled.
- **Running** – Containers are running on a node.
- **Succeeded** – Containers completed successfully.
- **Failed** – Containers terminated with an error.
- **Unknown** – Pod state can’t be determined.


## Cluster Creation Methods
Kubernetes clusters can be created in different ways depending on the environment:

| Tool/Service | Description                                              | Use Case                        |
| ------------ | -------------------------------------------------------- | ------------------------------- |
| **Minikube** | Runs Kubernetes locally inside a VM or Docker container. | Learning and local development. |
| **Kind**     | Runs Kubernetes inside Docker containers.                | CI/CD testing and quick setups. |
| **kubeadm**  | Bootstraps a Kubernetes cluster on your own machines.    | On-premises or cloud servers.   |
| **EKS**      | Managed Kubernetes by AWS.                               | Production workloads on AWS.    |
| **GKE**      | Managed Kubernetes by Google Cloud.                      | Production workloads on GCP.    |
| **AKS**      | Managed Kubernetes by Azure.                             | Production workloads on Azure.  |


## Introduction to Pods and Services

### Pods
A Pod is the smallest deployable unit in Kubernetes, encapsulating one or more containers with shared resources like storage and network.

- **Lifecycle**:
  - Pending → Running → Succeeded/Failed → Terminated

- **Use Cases**:
  - Running a single application container.
  - Running multiple containers that share resources and are tightly coupled (e.g., sidecar patterns).

### Services
Services provide stable networking and expose Pods to other applications or external traffic.

- **Types of Services**:
  1. **ClusterIP**: Exposes the service within the cluster.
  2. **NodePort**: Exposes the service on each node’s IP at a static port.
  3. **LoadBalancer**: Exposes the service to the internet using a cloud provider’s load balancer.

| Service Type     | Accessible From         | Typical Use Case                                | Example Access URL            |
| ---------------- | ----------------------- | ----------------------------------------------- | ----------------------------- |
| **ClusterIP**    | Inside cluster only     | Microservice-to-microservice communication      | `http://backend-service:8080` |
| **NodePort**     | External (via node IP)  | Testing or small setups without a load balancer | `http://<node-ip>:30080`      |
| **LoadBalancer** | Internet (via cloud LB) | Production applications accessible publicly     | `http://<public-ip>`          |

---

## Main Container and Sidecar Containers

### Main Container
The primary container that serves the main purpose of the application. Examples include application servers or web servers.

### Sidecar Container
An auxiliary container that provides supporting functionalities, such as logging, monitoring, or proxying.


## Run First Pod Using kubectl

1. **Create a Pod**:
   ```bash
   kubectl run nginx-pod --image=nginx --restart=Never
   ```
   - `--image`: Specifies the container image.
   - `--restart=Never`: Ensures the creation of a standalone Pod.

2. **Verify Pod**:
   ```bash
   kubectl get pods
   ```

3. **View Pod Details**:
   ```bash
   kubectl describe pod nginx-pod
   ```

---

## Expose Pod Using kubectl expose

1. **Expose Pod**:
   ```bash
   kubectl expose pod nginx-pod --type=NodePort --port=80
   ```
   - `--type=NodePort`: Exposes the Pod on a static port.
   - `--port`: Specifies the port the Pod listens on.

2. **Get Service Details**:
   ```bash
   kubectl get svc
   ```

3. **Access the Pod**:
   - Use the `<NodeIP>:<NodePort>` to access the exposed Pod.

---

## In-Depth: kubectl Usage

### Common Commands
- **View Resources**:
  ```bash
  kubectl get pods
  kubectl get svc
  kubectl get nodes
  ```

- **Delete Resources**:
  ```bash
  kubectl delete pod <pod-name>
  ```

- **Debugging**:
  ```bash
  kubectl logs <pod-name>
  kubectl exec -it <pod-name> -- /bin/bash
  ```

- Viewing Pod Events:
  ```bash
  kubectl describe pod <pod-name>
  ```

---
# Introduction to YAML Scripts and Kubernetes Manifest Files

## Understanding YAML Scripts
YAML (YAML Ain't Markup Language) is a human-readable data serialization format used extensively in Kubernetes for writing manifest files. It is used to define resources like Pods, Services, Deployments, and more.

### Key Features of YAML
- **Readable**: YAML is simple and easy to understand.
- **Indentation-Based**: Proper indentation is crucial.
- **Data Types**: Supports scalars (strings, numbers), lists, and dictionaries.

### Example YAML Structure
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

## Writing Manifest Files for Pods and Services
Kubernetes uses manifest files to describe the desired state of resources in the cluster. These files are written in YAML.

### Pod Manifest File
A Pod is the smallest deployable unit in Kubernetes.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  labels:
    app: my-app
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```
**Explanation**:
- `apiVersion`: The API version used (e.g., v1).
- `kind`: The type of resource (e.g., Pod).
- `metadata`: Metadata such as the name and labels.
- `spec`: Specification of the Pod, including containers and their properties.

### Service Manifest File
Services expose Pods to the network and enable communication between them.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-cl-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```
**Explanation**:
- `selector`: Matches the labels of the Pods to be exposed.
- `ports`: Defines the service port and the target port on the Pod.
- `type`: Specifies the service type (e.g., ClusterIP, NodePort)


---
## Kubernetes Networking: Intra-Pod and Inter-Pod Communication

Kubernetes networking is fundamental for ensuring smooth communication between various components, including pods, services, and external clients. It provides flexible networking configurations for intra-pod and inter-pod communication.

### Intra-Pod Communication
- **Definition**: Intra-pod communication refers to the communication between containers within the same pod.
- **Mechanism**: Containers in a pod share the same network namespace, which means they:
  - Share the same IP address.
  - Can communicate directly using `localhost` and exposed container ports.

### Inter-Pod Communication
- **Definition**: Inter-pod communication refers to the communication between pods.
- **Mechanism**:
  - Kubernetes assigns each pod a unique IP address.
  - Pods communicate directly using these IP addresses or via Kubernetes services.
  - Kubernetes ensures a flat network model where all pods can communicate without NAT.
 
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
    - name: nginx
      image: nginx:1.14.2
      ports:
        - containerPort: 80

    - name: java
      image: openjdk:17
      command: ["tail", "-f", "/dev/null"]
      ports:
        - containerPort: 8080
     

    - name: mysql
      image: mariadb:latest
      env:
        - name: MYSQL_ROOT_PASSWORD    
          value: redhat
      ports:
        - containerPort: 3306
```
### Kubernetes Service Types for Networking

#### 1. **ClusterIP**
- Default service type.
- Exposes the service only within the cluster.
- Example:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-clip-svc
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

#### 2. **NodePort**
- Exposes the service on a static port on each node.
- Example:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-np-svc
spec:
  type: NodePort
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
    nodePort: 31195
```

#### 3. **LoadBalancer**
- Exposes the service externally using a cloud provider’s load balancer.
- Example:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-lb-service
spec:
  type: LoadBalancer
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

## Key Networking Components

### 1. Pod IP
- Each pod gets a unique IP address within the cluster.
- Enables direct communication between pods without port conflicts.

### 2. Container Port
- The port exposed by the container inside the pod.
- Used for intra-pod communication.

### 3. Node IP
- IP address of the Kubernetes node.
- Used when accessing services exposed via NodePort or LoadBalancer.

### 4. Node Port
- A static port on the node that forwards traffic to the service.
- Example: Node IP + Node Port allows access to services from outside the cluster.

### 5. LoadBalancer
- Integrates with cloud provider load balancers to expose services externally.
- Automatically assigns external IPs for access.

## Examples

### Accessing a Pod Directly
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```
- Accessing directly via Pod IP:
  - `curl <POD_IP>:80`

### Accessing a Service via NodePort
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-np-service
spec:
  type: NodePort
  selector:
    app: my-pod
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30001
```
- Access:
  - `http://<NODE_IP>:30001`

### Accessing a Service via LoadBalancer
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-lb-service
spec:
  type: LoadBalancer
  selector:
    app: my-pod
  ports:
  - port: 80
    targetPort: 8080
```
- Access:
  - External IP provided by the load balancer.
  - `http://<EXTERNAL_IP>:80`

  ---

## ReplicationController and ReplicaSet
ReplicationControllers and ReplicaSets ensure that the specified number of Pod replicas are running at all times.

### ReplicationController
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: my-rc
spec:
  replicas: 3
  selector:
    app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: nginx
          image: nginx:latest
```
**Explanation**:
- `replicas`: Specifies the desired number of Pods.
- `selector`: Matches labels to identify Pods.
- `template`: Describes the Pods to be created.

### ReplicaSet
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-rs
spec:
  replicas: 3
  selector:
    matchExpressions:
    - {key: app, operator: NotIn, values: ["my-app"]}
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: nginx
          image: nginx:latest
```
**Explanation**:
- `matchLabels`: Provides a more flexible way of selecting Pods.

### Differences Between ReplicationController and ReplicaSet
| Feature               | ReplicationController                  | ReplicaSet                          |
|-----------------------|----------------------------------------|-------------------------------------|
| **Selector**          | Uses equality-based selectors only.   | Supports set-based and equality-based selectors. |
| **Flexibility**       | Limited in matching Pods.             | More flexible in selecting Pods based on labels. |
| **Usage**             | Older method, being phased out.       | Preferred in modern deployments.   |
---
## Deployments vs StatefulSets

### Deployments
A **Deployment** ensures a specified number of pod replicas are running at any given time. Deployments are best suited for stateless applications.

#### Features of Deployments
- Stateless nature, meaning all pods are interchangeable.
- Easy scaling and updates with zero downtime.
- Fast rollback capability.
- Pods are recreated with new identities upon termination.

#### Use Cases
- Web servers.
- APIs.
- Microservices with no data dependency.

### StatefulSets
A **StatefulSet** is used for applications requiring unique identities and persistent storage for each pod. These are suited for stateful applications.

#### Features of StatefulSets
- Maintains a stable identity for each pod.
- Supports persistent storage using PVCs (PersistentVolumeClaims).
- Ensures ordered deployment, scaling, and deletion of pods.
- Pod names are deterministic (e.g., `pod-0`, `pod-1`).

#### Use Cases
- Databases (e.g., MySQL, MongoDB).
- Distributed systems (e.g., Kafka, ZooKeeper).
- Applications requiring strict ordering.

#### Key Differences Between Deployments and StatefulSets
| Feature                | Deployment                 | StatefulSet              |
|------------------------|----------------------------|--------------------------|
| Nature                | Stateless                 | Stateful                |
| Pod Identity          | Interchangeable           | Unique and Stable       |
| Scaling Behavior      | Independent Pods          | Ordered Scaling         |
| Storage               | Non-persistent            | Persistent Volumes      |
| Update Strategy       | Rolling Updates           | Ordered Updates         |

---

## Stateless vs Stateful Applications

### Stateless Applications
- Do not retain data between sessions.
- Pods are interchangeable and can be terminated or replaced without affecting the application.
- Example: A web server serving static pages.

### Stateful Applications
- Require data persistence and unique identities.
- Replacement pods need to access the same persistent storage and retain their identities.
- Example: A database like PostgreSQL where data must be retained even if the pod restarts.

---

## Writing Manifests

### Manifest for a Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: my-deployment
    labels:
       app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      name: my-pod
      labels:
        app: my-app
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
  strategy:
    type: RollingUpdate
```

### Manifest for a StatefulSet
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
    name: my-sts
spec:
  selector:
    matchLabels:
        app: my-app
  serviceName: "my-sts-service"
  replicas: 3
  template:
    metadata:
      labels:
        app: my-app
    spec:
        containers:
           - name: mysql
             image: mysql:latest
             env:
              - name: MYSQL_ROOT_PASSWORD    
                value: redhat
             ports:
                - containerPort: 3306
```

---

## Deployment Strategies

### Recreate Strategy
- Terminates all existing pods before creating new ones.
- Suitable for non-critical updates.

### Rolling Update Strategy
- Updates pods incrementally.
- Ensures minimal downtime and availability during updates.

### Canary Deployment
- Deploys a small subset of new pods alongside existing ones to test the update.

### Blue-Green Deployment
- Creates a new set of pods ("blue") while the old set ("green") remains active, enabling a smooth transition.

---

## Understanding DaemonSets

A **DaemonSet** ensures that a copy of a specific pod runs on all or selected nodes within a cluster. DaemonSets are typically used for system-level applications.

### Features of DaemonSets
- Runs one pod per node.
- Automatically schedules pods on newly added nodes.
- Ensures uninterrupted system monitoring and logging.

### Use Cases
- Log collection (e.g., Fluentd, Logstash).
- Monitoring (e.g., Prometheus Node Exporter).
- Network plugins (e.g., Calico, Weave).

### Manifest for a DaemonSet
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
    name: my-dmnset
spec:
    selector:
      matchLabels:
          app: fluentd
    template:
      metadata:
        labels:
          app: fluentd
      spec:
        nodeSelector:
            hostname: node01
        containers:
          - name: fluentd
            image: fluentd:latest
            ports:
              - containerPort: 24224
```
# Kubernetes ConfigMap and Secret with Practical Steps

This guide explains how to use ConfigMap and Secret resources in Kubernetes to manage configuration data and sensitive information. It includes step-by-step instructions, practical examples, and detailed notes for students.

---

## 1. Introduction

### What is a ConfigMap?
A **ConfigMap** is a Kubernetes resource used to store non-confidential data as key-value pairs. It helps decouple configuration data from application code.

### What is a Secret?
A **Secret** is a Kubernetes resource designed to store confidential data, such as passwords or API keys, in a secure and encoded format. Secrets are encoded using Base64, providing a layer of obfuscation but not encryption.

### Why Use ConfigMap and Secret?
- **Separation of Concerns:** Decouples configuration from application logic.
- **Ease of Updates:** Configuration changes do not require rebuilding or redeploying applications.
- **Security:** Secrets ensure sensitive data is handled securely.

---

## 2. Practical Steps

### Step 1: Create the ConfigMap

#### Manifest File
Save the following content in a file named `config.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: my-cred
data:
  ROHIT: "redhat"
  USERNAME: "rohit"
  CITY: "pune"
```

#### Apply the ConfigMap
Run the following command to create the ConfigMap in your cluster:

```bash
kubectl apply -f config.yaml
```

#### Verify the ConfigMap
View the created ConfigMap:

```bash
kubectl get configmap my-cred
```
#### To describe  the configmap
```bash
kubectl describe configmap my-config
```
### Step 2: Create the Secret
**to create encrypted values**
```sh
echo -n "redhat" | base64
```
#### Manifest File
Save the following content in a file named `secret.yaml`:

```yaml
apiVersion: v1
kind: Secret
metadata:
    name: my-secret
data:
  ROHIT: "cmVkaGF0"
  USERNAME: "cm9oaXQ="
  CITY: "cHVuZQ=="
```

#### Apply the Secret
Run the following command to create the Secret in your cluster:

```bash
kubectl apply -f secret.yaml
```

#### Verify the Secret
To check the Secret:

```bash
kubectl get secret my-secret
```

Note: The values will appear base64-encoded. To decode, use:

```bash
echo "QWRtaW4=" | base64 --decode
```

---

## 3. Using ConfigMap and Secret in a Pod

### Example Pod Manifest

Create a file named `pod-with-config-and-secret.yaml` with the following content:

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: my-pod
spec:
   containers:
      - name: my-c
        image: mysql:latest
        ports:
          - containerPort: 3306
        env:
          - name: MYSQL_ROOT_PASSWORD
            valueFrom:
              secretKeyRef:
                name: my-secret
                key: ROHIT
 
          - name: USER_NAME
            valueFrom:
              secretKeyRef:
                name: my-secret
                key: USERNAME

          - name: MY_CITY
            valueFrom:
              secretKeyRef:
                name: my-secret
                key: CITY
```

### Apply the Pod Manifest

```bash
kubectl apply -f pod-with-config-and-secret.yaml
```

### Verify the Pod Environment Variables

```bash
kubectl exec pod-with-config-secret -- printenv | grep DB_
```

---

## 4. Notes and Best Practices

### ConfigMap Notes
1. **Non-Confidential Data:** ConfigMaps should not store sensitive data.
2. **Dynamic Updates:** ConfigMaps can be updated dynamically, and changes can reflect in running Pods if the configuration is mounted as a volume.
3. **Avoid Overloading:** Use ConfigMaps for lightweight configurations to prevent complexity.

### Secret Notes
1. **Secure Handling:** Avoid storing Secrets in plain text files. Use tools like `kubectl` to manage them.
2. **Encryption:** Enable encryption at rest for Secrets in your cluster for additional security.
3. **Access Control:** Use RBAC to restrict access to Secrets.

---
## 1. Introduction

### Persistent Volume (PV)
A **Persistent Volume** is a storage resource in a Kubernetes cluster that provides persistent storage, independent of Pod lifecycles. It is defined and managed by the cluster administrator.

### Persistent Volume Claim (PVC)
A **Persistent Volume Claim** is a request for storage by a user. Pods use PVCs to access PVs.

### Dynamic Provisioning
Dynamic provisioning automatically creates PVs based on a PVC when a StorageClass is specified. This is particularly useful for cloud-based storage systems like AWS EBS.

---



# **Kubernetes Persistent Volume (PV) and Persistent Volume Claim (PVC) Demo**

```sh
mkdir -p /mnt/data
```
```sh
df -hT
```

## **1. Persistent Volume (PV)**
### **Example: Persistent Volume (PV)**
```yaml
apiVersion: v1   # API version used for PersistentVolume resource
kind: PersistentVolume   # Declares this manifest as a PersistentVolume (PV)
metadata:
    name: my-vol   # Name of the PV (unique identifier in the cluster)

spec:
  capacity:
    storage: 5Gi   # The size of the PV (5 Gigabytes)

  volumeMode: Filesystem   # Specifies that the volume will be mounted as a filesystem (not raw block)

  accessModes:
      - ReadWriteOnce   # Only one node can mount this volume for read/write at a time

  persistentVolumeReclaimPolicy: Retain   # What happens when the PVC is deleted:
                                          # Retain → Keep the data
                                          # Delete → Remove the PV and data
                                          # Recycle → Wipe and make PV available again (deprecated)

  storageClassName: rohit   # StorageClass name associated with this PV (used for dynamic provisioning)

  local:
    path: /mnt/data   # Path on the node’s local filesystem where data will be stored

  nodeAffinity:   # Ensures the PV is only usable on specific nodes
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname   # Node label key used for selection
              operator: In                  # Operator meaning "must be in this list"
              values: ["controlplane"]      # Only the node with hostname "controlplane" can use this PV

```
 
 ## 2. Persistent Volume Claim (PVC)

 ```yaml
apiVersion: v1   # API version for PersistentVolumeClaim resource
kind: PersistentVolumeClaim   # Declares this manifest as a PVC
metadata:
    name: my-pvc   # Name of the PVC (unique identifier in the namespace)

spec:
  accessModes:
    - ReadWriteOnce   # Access mode: the PVC requires a volume that can be mounted read/write by only one node

  resources:
    requests:
      storage: 5Gi   # The PVC is requesting 5Gi of storage

  storageClassName: rohit   # Must match the PV's storageClassName so that it binds correctly

  volumeName: my-vol   # Explicitly binds this PVC to the PersistentVolume named "my-vol"

```
## 3. Pod Using PVC
```yaml
apiVersion: v1   # API version for Pod resource
kind: Pod        # Declares this manifest as a Pod
metadata:
  name: my-pod   # Name of the Pod

spec:
  containers:
    - name: nginx   # Container name inside the Pod
      image: nginx:latest   # Container image to run (latest Nginx)
      ports:
        - containerPort: 80   # Exposes port 80 (default HTTP port) inside the container

      volumeMounts:   # Defines where to mount volumes inside the container
        - mountPath: "/usr/share/nginx/html"   # Path inside container where volume will be mounted
          name: my-vol   # Must match the volume name defined below

  volumes:   # Volumes available to the Pod
    - name: my-vol   # Name of the volume (referenced above in volumeMounts)
      persistentVolumeClaim:
        claimName: my-pvc   # PVC name → binds this volume to the PVC "my-pvc"
 
```
## Testing Data Persistence
```
kubectl exec -it my-pod -- sh
echo "Persistent Storage Test!" > /usr/share/nginx/html/index.html
exit
kubectl delete pod my-pod
kubectl apply -f pod.yaml   #change the pod name
kubectl exec -it another-pod -- cat /usr/share/nginx/html/index.html
```
## 1. Types of AutoScaling in Kubernetes

### 1.1. Horizontal Pod Autoscaler (HPA)
- **Purpose**: Adjusts the number of Pod replicas in a deployment based on CPU, memory, or custom metrics.
- **Use Case**: Scaling out/in to handle variable workloads.

### 1.2. Vertical Pod Autoscaler (VPA)
- **Purpose**: Adjusts the resource requests and limits (CPU/memory) of containers in a Pod.
- **Use Case**: Ensures optimal resource utilization for Pods.

### 1.3. Cluster Autoscaler
- **Purpose**: Adjusts the number of nodes in a cluster based on pending Pods that cannot be scheduled.
- **Use Case**: Dynamically increases or decreases cluster size.

---

## 2. Practical Steps: Horizontal Pod Autoscaler (HPA)

### Step 1: Prerequisites

- Ensure the Kubernetes Metrics Server is installed:
  ```bash
  kubectl get deployment metrics-server -n kube-system
  ```
  If not installed, follow the [Metrics Server installation guide](https://github.com/kubernetes-sigs/metrics-server).

- Deploy an application (e.g., nginx) with resource requests and limits defined.

### Step 2: Create a Deployment

#### Manifest File
Save the following content in a file named `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "200Mi"
          limits:
            cpu: "200m"
            memory: "300Mi"

```

#### Apply the Deployment
```bash
kubectl apply -f deployment.yaml
```

### Step 3: Create the Horizontal Pod Autoscaler

#### Manifest File
Save the following content in a file named `hpa.yaml`:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```
#### Apply the HPA
```bash
kubectl apply -f hpa.yaml
```

**In killerkoda metrics-server is not installed so here are the commands to install metrics server on killercoda/minikube/kubeadm**
### ✅ How to Enable kubectl top pods in KillerKoda
**Install Metrics Server (clean install)**
```sh
kubectl delete -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml --ignore-not-found && \
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml && \
kubectl -n kube-system patch deployment metrics-server --type='json' -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/0","value":"--kubelet-insecure-tls"}]' && \
kubectl -n kube-system rollout restart deploy metrics-server
```
**to check cpu utilization of pods**
```sh
kubectl top pods
```



### Step 4: Verify the HPA

- Check the HPA status:
  ```bash
  kubectl get hpa
  ```

- Simulate a load test to trigger scaling:
  ```bash
  kubectl run -i --tty load-generator --image=busybox --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://nginx-deployment.default.svc.cluster.local; done"
  ```

- Observe the scaling behavior:
  ```bash
  kubectl get pods -w
  ```
  ## 📌 1. Introduction to Ingress
- **Ingress** is an API object that manages **external access** to services in a Kubernetes cluster.
- Provides:
  - **HTTP/HTTPS routing**
  - **Path-based routing** (`/app1` → Service1, `/app2` → Service2)
  - **Host-based routing** (`app1.example.com` → Service1, `app2.example.com` → Service2)
- Ingress requires an **Ingress Controller** (commonly **NGINX Ingress Controller**).

---

## 📌 2. Install NGINX Ingress Controller (Manifests)
Apply the official manifest:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```
**Verify installation:**

```sh
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```
### Sample Applications
✅ App1 (Deployment + Service)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
      - name: app1
        image: hashicorp/http-echo
        args:
        - "-text=Hello from App1"
        ports:
        - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: app1-svc
spec:
  selector:
    app: app1
  ports:
  - port: 80
    targetPort: 5678
```
### ✅ App2 (Deployment + Service)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app2
  template:
    metadata:
      labels:
        app: app2
    spec:
      containers:
      - name: app2
        image: hashicorp/http-echo
        args:
        - "-text=Hello from App2"
        ports:
        - containerPort: 5678
---
apiVersion: v1
kind: Service
metadata:
  name: app2-svc
spec:
  selector:
    app: app2
  ports:
  - port: 80
    targetPort: 5678
```
### Ingress Examples
**✅ Path-based Routing**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-based-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-svc
            port:
              number: 80
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2-svc
            port:
              number: 80
```
**✅ Host/Name-based Routing**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: host-based-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: app1.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-svc
            port:
              number: 80
  - host: app2.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app2-svc
            port:
              number: 80
```
**to ckech annotations**
```sh
kubectl describe ing path-based-ingress
```

