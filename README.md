# Kubernetes Learning Roadmap

> A structured roadmap to understand Kubernetes from fundamentals to production operations.

## How to Use This Roadmap

Learn every topic in three steps:

1. **Understand the concept**
2. **Practice it in a Kubernetes cluster**
3. **Break it intentionally and troubleshoot it**

Use one application as your continuous hands-on project while progressing through the roadmap.

---

# Phase 1 — Kubernetes Fundamentals

## 1. Kubernetes Architecture

- What is Kubernetes?
- Why Kubernetes?
- Kubernetes cluster
- Control Plane
- Worker Nodes
- `kube-apiserver`
- `etcd`
- `kube-scheduler`
- `kube-controller-manager`
- `kubelet`
- `kube-proxy`
- Container Runtime
- Kubernetes API
- Desired state vs current state
- Declarative vs imperative approach
- Reconciliation loop

### Practice

Inspect a local Minikube cluster and identify the major Kubernetes components.

---

## 2. Kubernetes Core Objects

Learn:

- Pod
- Namespace
- Labels
- Selectors
- Annotations
- ReplicaSet
- Deployment
- StatefulSet
- DaemonSet
- Job
- CronJob
- ServiceAccount
- ConfigMap
- Secret

Understand the relationship:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod
    ↓
Container
```

---

## 3. Pods

Learn:

- Pod lifecycle
- Pod phases
- Pod conditions
- Init containers
- Sidecar containers
- Multi-container Pods
- Restart policy
- Container states
- Liveness probe
- Readiness probe
- Startup probe
- Resource requests
- Resource limits
- QoS classes
- Pod eviction

### Important Concept

In production, you normally don't create standalone Pods directly. Higher-level controllers such as Deployments and StatefulSets manage them.

---

## 4. Deployments

Learn:

- Deployment
- ReplicaSet relationship
- `replicas`
- Scaling
- Rolling updates
- Rollbacks
- Revision history
- `maxSurge`
- `maxUnavailable`
- Deployment strategies

Example:

```yaml
spec:
  replicas: 3
```

Understand what happens when:

```text
3 → 5 replicas
5 → 2 replicas
```

Also practice updating an application and rolling it back.

---

# Phase 2 — Networking and Storage

## 5. Kubernetes Services

Learn:

- ClusterIP
- NodePort
- LoadBalancer
- ExternalName
- Service selectors
- Endpoints
- EndpointSlices
- Service discovery
- CoreDNS
- Pod IP
- Service IP
- Node IP
- Pod-to-Pod communication

Understand:

```text
User
  ↓
LoadBalancer
  ↓
Service
  ↓
Pod
```

---

## 6. Ingress

Learn:

- Ingress
- Ingress Controller
- NGINX Ingress
- Host-based routing
- Path-based routing
- TLS
- HTTPS
- Certificates
- IngressClass

Understand:

```text
api.example.com
       ↓
    Ingress
       ↓
 api-service
       ↓
    API Pods
```

### Practice

Route multiple applications using different hosts or paths.

---

## 7. Configuration Management

Learn:

- ConfigMap
- Secret
- Environment variables
- ConfigMap as volume
- Secret as volume
- External secret systems
- Secret management

### Practice

Deploy an application whose configuration can be changed without rebuilding the Docker image.

---

## 8. Storage

Learn:

- Volumes
- `emptyDir`
- `hostPath`
- PersistentVolume (PV)
- PersistentVolumeClaim (PVC)
- StorageClass
- Dynamic provisioning
- CSI
- Stateful application storage

Understand:

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
Storage
```

---

# Phase 3 — Workloads, Resources and Scaling

## 9. StatefulSet

Learn:

- Stable network identity
- Stable storage
- Ordered deployment
- Ordered termination
- Headless Services
- Database workloads
- Clustered applications

Understand why stateful workloads often use:

```text
StatefulSet + PVC
```

---

## 10. DaemonSet

Learn:

- One Pod per eligible node
- Node-level agents
- Logging agents
- Monitoring agents
- Rolling updates

Common examples:

```text
Node
 ├── Log Collector
 ├── Monitoring Agent
 └── Security Agent
```

---

## 11. Jobs and CronJobs

Learn:

- Run-to-completion workloads
- Job retries
- Backoff
- Parallel jobs
- Scheduled workloads
- Cleanup
- Job history

Examples:

```text
Job
 └── Database migration

CronJob
 └── Daily backup
```

---

## 12. Resource Management

Learn:

- CPU requests
- CPU limits
- Memory requests
- Memory limits
- QoS classes
  - Guaranteed
  - Burstable
  - BestEffort
- ResourceQuota
- LimitRange
- OOMKilled
- CPU throttling

---

## 13. Autoscaling

Learn:

- Horizontal Pod Autoscaler (HPA)
- Vertical Pod Autoscaler (VPA)
- Cluster Autoscaler
- Metrics Server
- CPU-based scaling
- Memory-based scaling
- Custom metrics

Understand:

```text
Traffic increases
       ↓
CPU increases
       ↓
HPA
       ↓
Pod count increases
```

---

# Phase 4 — Kubernetes Scheduling

## 14. Kubernetes Scheduler

Learn:

- Scheduler
- Scheduling process
- Node capacity
- Resource requests
- Scheduling constraints
- Scheduling decisions

Understand why Kubernetes chooses a particular node for a Pod.

---

## 15. Node Selection and Affinity

Learn:

- `nodeSelector`
- Node affinity
- Pod affinity
- Pod anti-affinity
- Topology spread constraints

Example:

```text
Node 1 → Database
Node 2 → Application
Node 3 → Monitoring
```

---

## 16. Taints and Tolerations

Learn:

- Taints
- Tolerations
- Dedicated nodes
- Preventing unwanted scheduling

Example:

```text
Database Node
     ↓
Taint
     ↓
Only Pods with matching toleration can run
```

---

## 17. Priority and Preemption

Learn:

- PriorityClass
- Pod priority
- Preemption
- Scheduling under resource pressure

---

# Phase 5 — Kubernetes Security

## 18. Authentication and Authorization

Learn:

- Authentication
- Authorization
- Kubernetes API access
- User identity
- Workload identity

Understand:

```text
User
 ↓
Authentication
 ↓
Authorization
 ↓
Kubernetes API
```

---

## 19. RBAC

Learn:

- Role
- ClusterRole
- RoleBinding
- ClusterRoleBinding
- Least privilege

Understand the difference between namespace-level and cluster-level permissions.

---

## 20. ServiceAccounts

Learn:

- ServiceAccount
- Pod identity
- API permissions
- Tokens
- Workload access to Kubernetes API

---

## 21. Workload Security

Learn:

- SecurityContext
- Run as non-root
- Linux capabilities
- Read-only root filesystem
- Pod Security Standards
- Privilege escalation controls

---

## 22. NetworkPolicy

Learn:

- Ingress rules
- Egress rules
- Namespace selectors
- Pod selectors
- IP blocks

Example:

```text
Frontend → Backend     ✅
Backend  → Database    ✅
Frontend → Database    ❌
Internet → Database    ❌
```

---

## 23. Image and Supply-Chain Security

Learn:

- Container image scanning
- Trivy
- Image provenance
- Image policies
- Secret management
- Runtime security

Tools to explore:

```text
Trivy
Vault
Falco
Kyverno
OPA
```

---

# Phase 6 — Helm and Advanced Kubernetes

## 24. Helm

Learn:

- Helm
- Helm Charts
- `Chart.yaml`
- `values.yaml`
- Templates
- `helm install`
- `helm upgrade`
- `helm rollback`
- `helm uninstall`
- Dependencies
- Environment-specific values

Understand:

```text
values.yaml
     ↓
Helm Templates
     ↓
Kubernetes YAML
     ↓
Kubernetes API
```

---

## 25. Custom Resources and Operators

Learn:

- CRD
- Custom Resource
- Controller pattern
- Operator pattern
- Admission Webhooks

Examples:

```text
Prometheus Operator
cert-manager
Argo CD
```

---

## 26. Kubernetes Extensibility

Learn:

- Admission Controllers
- Mutating Webhooks
- Validating Webhooks
- API Extensions
- Custom Controllers

---

# Phase 7 — Kubernetes Internals and Administration

## 27. Kubernetes API Request Flow

Understand:

```text
kubectl
   ↓
API Server
   ↓
Authentication
   ↓
Authorization
   ↓
Admission
   ↓
etcd
   ↓
Controllers
   ↓
Reconciliation
```

Be able to explain what happens internally when you execute:

```bash
kubectl apply -f deployment.yaml
```

---

## 28. Control Plane Internals

Learn:

- API Server
- etcd
- Scheduler
- Controller Manager
- Watch mechanism
- Control loops
- Reconciliation

---

## 29. Node Internals

Learn:

- kubelet
- Container Runtime
- CRI
- containerd
- CRI-O
- kube-proxy

Understand:

```text
Kubernetes
    ↓
kubelet
    ↓
CRI
    ↓
containerd / CRI-O
    ↓
Container
```

---

## 30. Kubernetes Networking Internals

Learn:

- CNI
- Pod networking
- Network plugins
- Service networking
- DNS
- Network namespaces
- Routing

Understand the role of CNI in providing Pod networking.

---

## 31. Kubernetes Storage Internals

Learn:

- CSI
- Volume lifecycle
- Dynamic provisioning
- Storage drivers
- Storage classes

Understand:

```text
Pod
 ↓
PVC
 ↓
StorageClass
 ↓
CSI Driver
 ↓
Cloud / Physical Storage
```

---

## 32. Cluster Administration

Learn:

- kubeadm
- Cluster installation
- Node joining
- Node removal
- Certificates
- etcd backup
- etcd restore
- Cluster upgrades
- Version compatibility

---

# Phase 8 — Troubleshooting and Production

## 33. Kubernetes Troubleshooting

Learn how to troubleshoot:

```text
Pending
CrashLoopBackOff
ImagePullBackOff
ErrImagePull
OOMKilled
CreateContainerConfigError
ContainerCreating
Terminating
NotReady
```

---

## 34. Essential kubectl Commands

Master:

```bash
kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs <pod> --previous
kubectl exec -it <pod> -- /bin/sh

kubectl get nodes
kubectl describe node <node>

kubectl get svc
kubectl get endpoints
kubectl get events

kubectl get deployments
kubectl get replicasets
kubectl get namespaces
```

### Recommended Troubleshooting Flow

```text
Application problem
       ↓
Check Deployment
       ↓
Check Pod
       ↓
Check Pod events
       ↓
Check container logs
       ↓
Check Service
       ↓
Check Endpoints
       ↓
Check Ingress
       ↓
Check Node
       ↓
Check Resources
```

---

## 35. Observability

Learn:

- Prometheus
- Grafana
- Metrics Server
- Logging
- ELK
- Loki
- OpenTelemetry
- Alerts
- Dashboards

Example:

```text
Kubernetes
    ↓
Prometheus
    ↓
Grafana
    ↓
Alerts
```

---

## 36. High Availability and Reliability

Learn:

- Multi-node clusters
- Control-plane HA
- Pod disruption
- Rolling deployments
- PodDisruptionBudget
- Backups
- Disaster Recovery
- Failure scenarios

---

## 37. Cloud Kubernetes

Learn:

### Amazon EKS

- Managed Kubernetes
- AWS Load Balancers
- IAM integration
- EBS/EFS storage

### Azure AKS

- Managed Kubernetes
- Azure Load Balancer
- Managed Identity
- Azure Disk / Azure Files

### Google GKE

- Managed Kubernetes
- Google Cloud networking
- IAM
- Persistent Disk

Also understand what the cloud provider manages versus what you manage.

---

## 38. CI/CD and GitOps

Learn:

```text
GitHub
   ↓
Azure DevOps / GitHub Actions
   ↓
Docker Build
   ↓
Container Registry
   ↓
Kubernetes
   ↓
Argo CD
```

Topics:

- CI/CD
- Docker image build
- Image registry
- Kubernetes deployment
- Helm
- Argo CD
- GitOps
- Environment promotion
- Rollback

---

## 39. Production Security

Learn:

- HashiCorp Vault
- Trivy
- Falco
- Kyverno
- OPA
- Image policies
- Secrets management
- Runtime security
- Network policies
- RBAC
- Pod security

---

## 40. Performance and Cost Optimization

Learn:

- Right-sizing CPU/memory
- Requests and limits
- Autoscaling
- Node sizing
- Bin packing
- Unused resource cleanup
- Cluster scaling
- Observability-driven optimization
- Cloud cost optimization

---

# Hands-On Project

Build one complete application while following the roadmap.

## Target Architecture

```text
                         GitHub
                            │
                            ▼
                  ┌──────────────────┐
                  │   CI/CD Pipeline │
                  │ Azure DevOps /   │
                  │ GitHub Actions   │
                  └────────┬─────────┘
                           │
                           ▼
                       Docker
                           │
                           ▼
                  Container Registry
                           │
                           ▼
                    Kubernetes
                           │
                     ┌─────┴─────┐
                     │  Ingress  │
                     └─────┬─────┘
                           │
                     ┌─────┴─────┐
                     │ Services  │
                     └─────┬─────┘
                           │
                ┌──────────┴──────────┐
                ▼                     ▼
            Frontend               Backend
                                      │
                                      ▼
                                  Database
                                      │
                                      ▼
                                     PVC

              Prometheus ───────► Grafana
                   │
                   ▼
                 Alerts

                Argo CD
                   │
                   ▼
                GitOps
```

---

# What You Should Be Able to Explain

After completing this roadmap, you should be able to answer:

1. What happens internally when `kubectl apply -f deployment.yaml` is executed?
2. How does a Deployment create and maintain Pods?
3. How does a ReplicaSet maintain the desired number of Pods?
4. How does a Service find the correct Pods?
5. How does DNS work inside Kubernetes?
6. How does traffic reach a Pod through Ingress?
7. What happens when a Pod crashes?
8. Why does a Pod enter `CrashLoopBackOff`?
9. Why is a Pod stuck in `Pending`?
10. How does Kubernetes choose a node for a Pod?
11. What is the difference between requests and limits?
12. When should you use Deployment vs StatefulSet vs DaemonSet?
13. How does Kubernetes provide persistent storage?
14. What are PV, PVC and StorageClass?
15. How does RBAC protect the Kubernetes API?
16. How does HPA scale an application?
17. What are CNI, CSI and CRI?
18. What is the role of etcd?
19. What does kubelet do?
20. How would you troubleshoot an application that is running but unreachable?

---

# Recommended Learning Order

| Stage | Focus | Outcome |
|---|---|---|
| 1 | Architecture, Pods, Deployments | Understand Kubernetes object model |
| 2 | Services, DNS, Ingress | Understand application networking |
| 3 | Config, Storage, Stateful workloads | Run stateful applications |
| 4 | Resources, Scheduling, Autoscaling | Operate workloads efficiently |
| 5 | Security, RBAC, NetworkPolicy | Secure workloads |
| 6 | Helm, CRDs, Operators, Internals | Understand advanced Kubernetes |
| 7 | Troubleshooting, Monitoring | Diagnose production problems |
| 8 | Cloud, GitOps, HA, DR | Operate production clusters |

---

# Final Goal

The goal is **not** simply to memorize `kubectl` commands.

You should eventually be able to:

- Design a Kubernetes architecture
- Deploy applications
- Configure networking
- Manage storage
- Secure workloads
- Configure RBAC
- Scale applications
- Schedule workloads
- Monitor applications
- Troubleshoot failures
- Build CI/CD pipelines
- Implement GitOps
- Manage Helm deployments
- Operate EKS/AKS/GKE
- Perform backups and disaster recovery
- Explain Kubernetes internals
- Optimize performance and cost

> **If you can build, deploy, break, troubleshoot, secure, monitor, scale, and explain one complete Kubernetes application, you will have a strong practical Kubernetes foundation.**
