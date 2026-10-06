# Phase 1.2 — Kubernetes Core Objects

## 1. What is a Kubernetes Object?

A Kubernetes object is a persistent representation of the desired state of something in the cluster.

Common objects include:

- Pod
- Deployment
- ReplicaSet
- Service
- Namespace
- ConfigMap
- Secret
- StatefulSet
- DaemonSet
- Job
- CronJob
- ServiceAccount

Most objects contain:

```yaml
apiVersion:
kind:
metadata:
spec:
```

Some also expose runtime information in:

```yaml
status:
```

---

# 2. apiVersion

`apiVersion` identifies the Kubernetes API group and version used by the object.

Examples:

```yaml
apiVersion: v1
```

```yaml
apiVersion: apps/v1
```

Examples:

- Pod → `v1`
- Service → `v1`
- Deployment → `apps/v1`
- StatefulSet → `apps/v1`

---

# 3. kind

`kind` tells Kubernetes what type of object you are creating.

```yaml
kind: Pod
```

or:

```yaml
kind: Deployment
```

---

# 4. metadata

Metadata identifies and describes an object.

Example:

```yaml
metadata:
  name: nginx
  namespace: learning
  labels:
    app: nginx
```

Common metadata:

- name
- namespace
- labels
- annotations
- UID
- creation timestamp

---

# 5. Labels

Labels are key-value pairs attached to objects.

```yaml
labels:
  app: nginx
  environment: production
```

Labels are extremely important because Kubernetes uses them to identify groups of objects.

---

# 6. Selectors

Selectors find objects based on labels.

Example:

```yaml
selector:
  matchLabels:
    app: nginx
```

This means:

> Select objects whose label `app` equals `nginx`.

A Service may use:

```yaml
selector:
  app: nginx
```

to find Pods with:

```yaml
labels:
  app: nginx
```

---

# 7. Labels vs Annotations

## Labels

Used for identification and selection.

```yaml
labels:
  app: backend
  environment: prod
```

## Annotations

Used for additional metadata that is generally not used for selection.

```yaml
annotations:
  description: "Backend application"
```

Ingress controllers, monitoring systems, and other Kubernetes integrations often use annotations.

---

# 8. Namespace

Namespaces provide logical separation inside a cluster.

Example:

```text
Cluster
├── development
├── testing
├── staging
└── production
```

Create:

```bash
kubectl create namespace learning
```

Use:

```bash
kubectl get pods -n learning
```

---

# 9. Pod

A Pod is the smallest deployable unit in Kubernetes.

A Pod contains one or more containers that share:

- Network namespace
- Pod IP
- Volumes
- Lifecycle

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.27
```

---

# 10. ReplicaSet

A ReplicaSet maintains a desired number of matching Pods.

Example:

```yaml
spec:
  replicas: 3
```

Conceptually:

```text
ReplicaSet
   │
   ├── Pod 1
   ├── Pod 2
   └── Pod 3
```

If Pod 2 disappears:

```text
ReplicaSet
   │
   ├── Pod 1
   ├── Pod 3
   └── New Pod
```

In most application deployments, you manage ReplicaSets indirectly through Deployments.

---

# 11. Deployment

Deployment manages application releases and ReplicaSets.

```text
Deployment
     ↓
ReplicaSet
     ↓
Pods
```

Deployment provides:

- Rolling updates
- Rollbacks
- Scaling
- Replica management
- Revision history

---

# 12. Service

A Service provides a stable network endpoint for a changing set of Pods.

Pods are ephemeral.

Their IP addresses can change.

A Service provides stable access.

```text
Client
  ↓
Service
  ↓
Pod
```

---

# 13. ConfigMap

ConfigMap stores non-sensitive configuration.

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_MODE: production
  LOG_LEVEL: info
```

Use it as environment variables or files.

---

# 14. Secret

Secret is intended for sensitive configuration.

Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: database-secret
type: Opaque
stringData:
  username: appuser
  password: change-me
```

Important:

Kubernetes Secrets provide a Kubernetes object for sensitive data, but you should not assume that simply using a Secret automatically solves every secret-management problem. Production environments often integrate external secret managers such as Vault or cloud secret services.

---

# 15. StatefulSet

StatefulSet is designed for stateful workloads that require stable identity and/or stable storage.

Examples:

- Databases
- Distributed systems
- Stateful clusters

Pods can have stable names:

```text
database-0
database-1
database-2
```

---

# 16. DaemonSet

DaemonSet ensures that a Pod runs on each eligible node.

Common uses:

- Log collection
- Monitoring agents
- Security agents
- Node-level networking components

Example:

```text
Node 1 → Agent
Node 2 → Agent
Node 3 → Agent
```

---

# 17. Job

A Job runs a task until completion.

Example:

```text
Job
 ↓
Migration
 ↓
Completed
```

Jobs are useful for:

- Database migrations
- Batch processing
- One-time tasks

---

# 18. CronJob

CronJob creates Jobs according to a schedule.

Example:

```yaml
spec:
  schedule: "0 2 * * *"
```

This represents a scheduled task and should be interpreted according to standard cron syntax.

Common use cases:

- Backups
- Reports
- Cleanup
- Scheduled processing

---

# 19. ServiceAccount

ServiceAccounts provide an identity for workloads.

Example:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
```

Pods can use:

```yaml
spec:
  serviceAccountName: app-sa
```

Permissions are then controlled through RBAC.

---

# 20. Object Relationships

The most important relationships to understand:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
```

```text
Service
    ↓
Label Selector
    ↓
Pods
```

```text
Ingress
    ↓
Service
    ↓
Pods
```

```text
StatefulSet
    ↓
Stateful Pods
    ↓
PVC
    ↓
Storage
```

---

# 21. Complete Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
  namespace: learning
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: web
  namespace: learning
spec:
  selector:
    app: web
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
```

Notice:

```text
Deployment
    ↓
Pods labeled app=web
    ↓
Service selector app=web
    ↓
Service sends traffic to Pods
```

---

# 22. Useful Commands

```bash
kubectl get all
kubectl get pods
kubectl get deployments
kubectl get replicasets
kubectl get services
kubectl get namespaces
kubectl get configmaps
kubectl get secrets
```

Detailed YAML:

```bash
kubectl get deployment web -o yaml
```

Describe:

```bash
kubectl describe deployment web
```

---

# 23. Hands-On Exercise

Create a namespace:

```bash
kubectl create namespace learning
```

Create a Deployment:

```bash
kubectl create deployment nginx \
  --image=nginx:1.27 \
  -n learning
```

Scale:

```bash
kubectl scale deployment nginx --replicas=3 -n learning
```

Inspect:

```bash
kubectl get deployment -n learning
kubectl get rs -n learning
kubectl get pods -n learning
```

Ask yourself:

> Which object actually created the Pods?

Answer:

```text
Deployment
   ↓
ReplicaSet
   ↓
Pods
```

---

# 24. Interview Questions

1. What is a Kubernetes object?
2. What is `apiVersion`?
3. What is `kind`?
4. What is metadata?
5. What are labels?
6. What are selectors?
7. Labels vs annotations?
8. What is a Namespace?
9. What is a Pod?
10. What is a ReplicaSet?
11. What is a Deployment?
12. What is a Service?
13. Deployment vs StatefulSet?
14. Deployment vs DaemonSet?
15. Job vs CronJob?
16. ConfigMap vs Secret?
17. What is a ServiceAccount?
18. How does a Service find Pods?
19. Why do we normally manage Pods through controllers?

---

# 25. Must Understand

Draw these relationships without looking at notes:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pod
```

```text
Service
    ↓
Selector
    ↓
Pod labels
```

```text
Ingress
    ↓
Service
    ↓
Pods
```

Once these relationships are clear, Kubernetes becomes much easier to reason about.
