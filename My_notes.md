# EKS Ingress, Storage, Networking, and RBAC – Notes

## EKS Ingress: IP vs Host (Target Types)

Ingress defines how external traffic is routed to services inside the Kubernetes cluster.

- **Target types**:
  - **IP target type**
  - **Instance target type**

Ingress routing can be defined using:
- **Path-based routing**
- **Host-based routing (domain/subdomain)**

If we have **paths**, we use **path-based routing**.  
If we have **domain name / subdomain name**, we use **host-based routing**.

We need Ingress for path-based routing because:
- A `Service` of type `LoadBalancer` creates **one load balancer per service**
- If we use a LoadBalancer for every microservice, it will create a new load balancer each time
- This is **costly**

So, we use **Ingress** for:
- Path-based routing  
- Host-based routing  

Ingress acts as a **single entry point**.

### Why we need Ingress in Kubernetes

We need Ingress in a Kubernetes cluster to act as a **smart, centralized entry point** for external web traffic (HTTP/HTTPS).

---

## EKS Storage: StorageClass, PV, and PVC

### Persistent Volume (PV)

- **What it is**: The actual storage in the cluster (like a hard disk)
- **Created by**:
  - Admin (static provisioning)
  - Dynamically via StorageClass
- **Scope**: Cluster-wide (any pod can use it if allowed)
- **Represents**: Physical storage resource (EBS, NFS, etc.)

---

## Types of Storage Provisioning in Kubernetes

There are **2 types of provisioning storage into Kubernetes**:

### 1. Static Provisioning

- We manually create an **EBS volume**
- Then we create:
  - PV YAML
  - PVC
- Pod uses the PVC

### 2. Dynamic Provisioning (Always Recommended)

Steps:
1. Create a **StorageClass** (e.g., `ebs-gp3`)
2. Create a **PVC** requesting storage (e.g., 10Gi) and reference the StorageClass
3. Kubernetes + AWS CSI driver automatically:
   - Creates the EBS volume
   - Creates the PV
   - Binds PV to PVC
4. Pod uses the PVC → dynamically provisioned EBS

Notes:
- We create the **StorageClass**
- It will create the **EBS & PV**
- After that, **PVC** requests the storage
- **No PV is defined manually**
- Kubernetes automatically creates a PV using this StorageClass

---

## StorageClass Example

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-gp3
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  fsType: ext4
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

---

## PersistentVolumeClaim (PVC) Example

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-app-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: ebs-gp3
```

---

## Pod Using PVC Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
    - name: nginx-container
      image: nginx
      volumeMounts:
        - name: app-storage
          mountPath: "/usr/share/nginx/html"
  volumes:
    - name: app-storage
      persistentVolumeClaim:
        claimName: my-app-pvc
```

---

## Network Isolation in Kubernetes

To deny **cross-namespace communication** in Kubernetes, you use **NetworkPolicies**.

- This is the **standard and correct way** to achieve network isolation.

---

## CNI in Amazon EKS

**CNI** stands for **Container Network Interface**.

In Amazon EKS, CNI is responsible for:
- Assigning IP addresses to Pods
- Routing traffic
- Pod-to-Pod and Pod-to-external communication

---

## Network Card (NIC)

A **Network Card** (also called **NIC – Network Interface Card**) is hardware that allows a computer/server to connect to a network.

---

## How Amazon VPC CNI Works

Each worker node (EC2) has:
- An **Elastic Network Interface (ENI)**
- Multiple **secondary private IPs**

When a Pod is created:
- CNI assigns **one VPC IP** to the Pod from the node’s ENI
- Pod gets an IP like `10.0.1.25`

---

## NetworkPolicy: Default Deny (Backend Namespace)

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: backend
spec:
  podSelector: {}
  policyTypes:
    - Ingress
```

---

## Allow Frontend Namespace to Access Backend

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: backend
spec:
  podSelector: {}
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: frontend
```

---

## RBAC in Kubernetes

### What is a ClusterRole?

A **ClusterRole** defines:
- What actions are allowed
- On which resources
- Across all namespaces or cluster-level resources

**Example**: Read-only access to pods in all namespaces

---

### What is a ClusterRoleBinding?

A **ClusterRoleBinding**:
- Assigns a ClusterRole
- To a user / group / serviceAccount
- Across the whole cluster
