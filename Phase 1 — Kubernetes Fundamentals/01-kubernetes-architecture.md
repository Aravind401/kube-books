# Phase 1.1 — Kubernetes Architecture

## 1. What is Kubernetes?

Kubernetes is an open-source platform for deploying, scaling, managing, and automating containerized applications.

A container runtime can run a container, but Kubernetes manages containers as part of a larger system.

### Without Kubernetes

```text
Developer
   ↓
Docker image
   ↓
Run container
   ↓
Manually manage:
- Restart
- Scaling
- Networking
- Deployment
- Updates
- Service discovery
```

### With Kubernetes

```text
Developer
   ↓
Container Image
   ↓
Kubernetes
   ├── Scheduling
   ├── Scaling
   ├── Self-healing
   ├── Networking
   ├── Service discovery
   ├── Rolling updates
   └── Desired-state management
```

---

# 2. Kubernetes Cluster

A Kubernetes cluster consists mainly of:

```text
                 Kubernetes Cluster
                       │
          ┌────────────┴────────────┐
          │                         │
     Control Plane              Worker Nodes
          │                         │
    ┌─────┼─────┐             ┌─────┴─────┐
    │     │     │             │           │
 API   Scheduler Controller  Node 1      Node 2
Server          Manager        │           │
    │                           Pods        Pods
   etcd
```

## Control Plane

The control plane makes decisions about the cluster.

It manages:

- Cluster state
- Scheduling
- Controllers
- API requests
- Persistent cluster data

## Worker Node

A worker node runs application workloads.

It normally contains:

- kubelet
- kube-proxy
- Container runtime
- Pods

---

# 3. Control Plane Components

## 3.1 kube-apiserver

The Kubernetes API Server is the central entry point to the Kubernetes API.

Almost every Kubernetes operation goes through it.

Example:

```bash
kubectl get pods
```

The flow is approximately:

```text
kubectl
   ↓
API Server
   ↓
Authenticate
   ↓
Authorize
   ↓
Admission
   ↓
Read cluster state
```

The API server:

- Receives API requests
- Authenticates clients
- Authorizes requests
- Runs admission controls
- Validates API objects
- Communicates with etcd
- Exposes the Kubernetes API

### Important

`kubectl` does not directly communicate with kubelet to create a Deployment.

It normally communicates with the API Server.

---

# 4. etcd

`etcd` is the distributed key-value store used by Kubernetes to store cluster state.

It contains information such as:

- Deployments
- Pods
- Services
- ConfigMaps
- Secrets
- Nodes
- Namespaces
- Cluster configuration

Conceptually:

```text
Kubernetes API Server
        ↓
      etcd
        ↓
Persistent cluster state
```

### Important

etcd is not a normal application database such as PostgreSQL.

It is designed for Kubernetes control-plane state.

---

# 5. kube-scheduler

The scheduler decides which worker node should run a newly created Pod.

Example:

```text
Pod needs to run
       ↓
Scheduler
       ↓
Evaluate nodes
       ↓
Node 1 ❌ insufficient resources
Node 2 ✅ suitable
Node 3 ❌ scheduling constraint
       ↓
Pod assigned to Node 2
```

The scheduler considers factors such as:

- CPU requests
- Memory requests
- Node availability
- nodeSelector
- Affinity
- Anti-affinity
- Taints
- Tolerations
- Topology constraints
- Pod priority

### Important

The scheduler chooses the node.

The kubelet on that node actually starts the containers.

---

# 6. kube-controller-manager

Kubernetes contains many controllers.

A controller continuously compares:

```text
Desired State
     vs
Current State
```

and attempts to make current state match desired state.

Example:

```yaml
spec:
  replicas: 3
```

Suppose only two Pods exist:

```text
Desired = 3
Current = 2
```

The controller notices the difference and creates another Pod.

```text
Desired: 3
Current: 2
    ↓
Controller
    ↓
Create Pod
    ↓
Current: 3
```

This is the reconciliation loop.

---

# 7. kubelet

The kubelet runs on every worker node.

Its responsibility is to make sure Pods assigned to its node are running correctly.

Conceptually:

```text
API Server
    ↓
Pod assigned to Node 1
    ↓
kubelet on Node 1
    ↓
Container Runtime
    ↓
Container
```

The kubelet:

- Watches Pod specifications
- Creates containers through the container runtime
- Performs health checks
- Restarts containers when required
- Reports node and Pod status
- Mounts volumes
- Manages Pod lifecycle

---

# 8. Container Runtime

Kubernetes needs a container runtime to actually execute containers.

Common runtimes include:

- containerd
- CRI-O

Kubernetes communicates with the runtime through the Container Runtime Interface (CRI).

```text
kubelet
   ↓
CRI
   ↓
containerd / CRI-O
   ↓
Container
```

---

# 9. kube-proxy

kube-proxy is associated with Kubernetes Service networking on nodes.

It helps implement Service traffic routing using networking mechanisms available on the node.

Conceptually:

```text
Client
  ↓
Service IP
  ↓
Node networking
  ↓
Selected Pod
```

Modern Kubernetes networking can use different implementations and dataplanes, so kube-proxy should be understood as part of the Service networking model rather than as the entire networking system.

---

# 10. Kubernetes API

Kubernetes is API-driven.

Resources such as:

- Pods
- Deployments
- Services
- ConfigMaps
- Secrets

are represented as API objects.

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
```

You submit this object to the API Server.

```bash
kubectl apply -f pod.yaml
```

---

# 11. Desired State vs Current State

This is one of the most important Kubernetes concepts.

Suppose you specify:

```yaml
spec:
  replicas: 3
```

You are telling Kubernetes:

> I want three replicas.

That is the desired state.

If only two are running:

```text
Desired State = 3
Current State = 2
```

Kubernetes works to reconcile the difference.

---

# 12. Declarative vs Imperative

## Imperative

You tell Kubernetes exactly what action to perform.

Example:

```bash
kubectl create deployment nginx --image=nginx
```

You are instructing Kubernetes:

> Create this Deployment.

## Declarative

You describe what the final state should be.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
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
          image: nginx:latest
```

Then:

```bash
kubectl apply -f deployment.yaml
```

Kubernetes determines what actions are necessary.

### Key idea

Declarative configuration describes:

```text
WHAT I WANT
```

rather than:

```text
EXACTLY HOW TO DO IT
```

---

# 13. Reconciliation Loop

The reconciliation loop is fundamental to Kubernetes.

```text
          Desired State
                │
                ▼
          Controller
                │
                ▼
          Current State
                │
                ▼
         Compare states
                │
          ┌─────┴─────┐
          │ Different?│
          └─────┬─────┘
                Yes
                 ↓
          Take corrective action
                 ↓
            New state
                 ↓
              Repeat
```

This is why Kubernetes can be self-healing.

---

# 14. What Happens During kubectl apply?

Suppose:

```bash
kubectl apply -f deployment.yaml
```

A simplified flow is:

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
Object validation
   ↓
Store desired state
   ↓
Controller observes object
   ↓
ReplicaSet created/updated
   ↓
Pods created
   ↓
Scheduler selects nodes
   ↓
kubelet sees assigned Pods
   ↓
Container runtime starts containers
   ↓
Status reported back
```

This sequence is extremely important for interviews and troubleshooting.

---

# 15. Useful Commands

```bash
kubectl cluster-info
kubectl get nodes
kubectl get pods -A
kubectl get namespaces
kubectl get componentstatuses
```

For modern Kubernetes versions, some older component-status commands may not be available or recommended. Prefer inspecting actual control-plane components and cluster health using your distribution's supported mechanisms.

Inspect configuration:

```bash
kubectl config view
kubectl config get-contexts
kubectl config current-context
```

Inspect API resources:

```bash
kubectl api-resources
kubectl api-versions
```

---

# 16. Hands-On Exercise

## Exercise 1 — Inspect the Cluster

Run:

```bash
kubectl get nodes
kubectl get pods -A
kubectl get namespaces
```

Questions:

1. How many nodes exist?
2. Which node is the control plane?
3. Which system Pods are running?
4. Which namespaces exist?

## Exercise 2 — Create a Namespace

```bash
kubectl create namespace learning
kubectl get namespaces
```

## Exercise 3 — Observe Kubernetes

Create a Deployment:

```bash
kubectl create deployment nginx --image=nginx
```

Then:

```bash
kubectl get deployment
kubectl get replicasets
kubectl get pods
```

Notice:

```text
Deployment
   ↓
ReplicaSet
   ↓
Pod
```

---

# 17. Interview Questions

1. What is Kubernetes?
2. What is a Kubernetes cluster?
3. What does the API Server do?
4. What is etcd?
5. What does the scheduler do?
6. What does kubelet do?
7. What is kube-proxy?
8. What is a container runtime?
9. What is the difference between desired and current state?
10. What is reconciliation?
11. What is declarative configuration?
12. What happens when `kubectl apply` is executed?
13. How does Kubernetes self-heal?
14. What is the difference between control plane and worker nodes?

---

# 18. Must Understand Before Moving On

You should be able to draw this from memory:

```text
                    Kubernetes Cluster

                 ┌─────────────────────┐
                 │    Control Plane    │
                 │                     │
kubectl ────────►│ API Server          │
                 │     │               │
                 │   etcd              │
                 │     │               │
                 │ Scheduler            │
                 │ Controller Manager   │
                 └─────────┬───────────┘
                           │
                  Pod scheduling
                           │
            ┌──────────────┴──────────────┐
            │                             │
       Worker Node                    Worker Node
       ┌────────────┐                 ┌────────────┐
       │ kubelet    │                 │ kubelet    │
       │ kube-proxy │                 │ kube-proxy │
       │ runtime    │                 │ runtime    │
       │ Pod        │                 │ Pod        │
       └────────────┘                 └────────────┘
```

If you understand this diagram, you have the foundation for the rest of Kubernetes.
