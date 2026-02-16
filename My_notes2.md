# Kubernetes & Amazon EKS – Core Concepts and Commands

This document explains key Kubernetes and Amazon EKS concepts including
Node Groups, Cluster Autoscaler, cluster switching commands, node maintenance,
Amazon ECR authentication, and EKS Fargate.

---

## 1. Node Group vs Cluster Autoscaler

### Key Differences

| Feature | Node Group | Cluster Autoscaler |
|------|----------|------------------|
| What it is | Infrastructure definition | Kubernetes controller |
| Creates nodes | Yes (via Auto Scaling Group) | No |
| Decides when to scale | Manual or ASG policies | Based on pending pods |
| Works at pod level | No | Yes |
| Required for autoscaling | Yes | Yes |
| Cloud-aware | Yes | Yes |

---

## 2. How Node Groups and Cluster Autoscaler Work Together

1. A Pod is created  
2. Kubernetes Scheduler tries to place the Pod  
3. No node has sufficient CPU or memory  
4. Pod remains in Pending state  
5. Cluster Autoscaler detects the pending Pod  
6. Cluster Autoscaler increases Node Group desired capacity  
7. Auto Scaling Group launches a new EC2 node  
8. Node joins the cluster  
9. Pod is scheduled and starts running  

---

## 3. Switching Between Kubernetes Clusters

### Switch Cluster Context
```bash
kubectl config use-context <context-name>
```
Switches between different Kubernetes clusters or contexts.

---

## 4. Node Maintenance Commands

### Cordon a Node
```bash
kubectl cordon <node-name>
```
Marks a node as unschedulable.

### Drain a Node
```bash
kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
```
Safely evicts all pods from a node.

### Uncordon a Node
```bash
kubectl uncordon <node-name>
```
Marks the node as schedulable again.

---

## 5. Amazon ECR Login Command

```bash
aws ecr get-login-password --region <region> \
| docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
```

Used before pushing or pulling images from Amazon ECR.

---

## 6. Amazon EKS Fargate

### What is EKS Fargate?

Amazon EKS Fargate allows running Kubernetes Pods without managing EC2 nodes.
AWS manages compute infrastructure automatically.

---

### Key Characteristics

| Feature | EKS Fargate |
|------|-----------|
| Node management | Not required |
| Scaling | Automatic per pod |
| Pricing | Pay per pod CPU and memory |
| Security | Pod-level isolation |
| Best for | Microservices, batch jobs |

---

### How Fargate Works

1. Pod is created  
2. Pod matches a Fargate profile  
3. AWS provisions compute automatically  
4. Pod runs without visible worker nodes  

---

### When to Use Fargate

- No node management required
- Low operational overhead
- Event-driven or burst workloads
- Secure workloads with isolation

---

### When Not to Use Fargate

- GPU workloads
- DaemonSets
- High-performance networking
- Custom OS configurations

---

## 7. Node Groups vs Fargate

| Feature | Node Group | Fargate |
|------|----------|--------|
| EC2 management | Required | Not required |
| DaemonSets | Supported | Not supported |
| Cost control | Flexible | Simpler, may cost more |
| Operational overhead | Higher | Very low |

---

## 8. Jira Overview

Jira is a project management tool used to plan, track, and manage software development and DevOps workflows.

---

## 9. Summary

- Node Groups provide compute infrastructure
- Cluster Autoscaler controls scaling decisions
- Fargate removes node management
- Node maintenance ensures cluster stability
- ECR authentication is required for container workflows

---

## 10. Recommended Strategy

- Use Node Groups with Cluster Autoscaler for core workloads
- Use Fargate for lightweight or burst workloads
- Combine both in one EKS cluster when required

---
