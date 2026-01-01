EKS Ingress 
ip vs host --> are the target types --> here how it defines the routes of traffic
like if wee have the the paths we will use IP target type
& if we have Domain name/subdomain name we will use --> Hostname

we need ingress for path based routing because load balancer service type it give DNS 
if we use load balancer for every microservice it will create everytime a new load balancer so it will be costly
so that we need ingress for path based routing or host based routing

We need
Ingress in a Kubernetes cluster to act as a smart, centralized entry point for external web traffic (HTTP/HTTPS)

Eks storage class and pv & pvc 

Persistent Volume

What it is: The actual storage in the cluster (like a hard disk).
Created by: Admin (static) or dynamically via StorageClass.
Scope: Cluster-wide (any pod can use it if allowed).
Represents: Physical storage resource (EBS, NFS, etc.)


There are 2 types of provisioning storage into k8s

STatic 
we create EBS & then we create PV yml and then pvc

Dynamic (always recommanded)

Create StorageClass (e.g., ebs-gp3)
Create PVC requesting 10Gi and reference StorageClass
Kubernetes + AWS CSI driver automatically creates EBS volume
PV is created and bound to PVC
Pod uses PVC → dynamically provisioned EBS


We create storage class it will create the EBS & PV and after that PVC (request for the storage)
No PV is defined manually. Kubernetes will automatically create a PV using this StorageClass.

storage-class
```bash
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

pvc
```bash
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

Pod that use pvc
```bash
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
-------------------------------------------------------
To deny cross-namespace communication in Kubernetes, you use NetworkPolicies.
This is the standard and correct way to achieve network isolation.

CNI stands for Container Network Interface.
In Amazon EKS, CNI is responsible for how Pods get networking—IP addresses, routing, and communication inside and outside the Kubernetes cluster.

what is a Network Card (NIC)?
A Network Card (also called NIC – Network Interface Card) is hardware that allows a computer/server to connect to a network.

How Amazon VPC CNI works

Each worker node (EC2) has:
An Elastic Network Interface (ENI)
Multiple secondary private IPs

When a Pod is created:
CNI assigns one VPC IP to the Pod from the node’s ENI
Pod gets an IP like 10.0.1.25

Default deny for backend(example) namespace

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
  namespace: backend
spec:
  podSelector: {}
  policyTypes:
  - Ingress


Allow frontend namespace to backend
```bash
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
-------------------------------------------

What is a ClusterRole?

A ClusterRole defines:
What actions are allowed
On which resources
Across all namespaces or cluster-level resources

Example: Read-only access to pods in all namespaces


What is a ClusterRoleBinding?

A ClusterRoleBinding:
Assigns a ClusterRole
To a user / group / serviceAccount
Across the whole cluster






