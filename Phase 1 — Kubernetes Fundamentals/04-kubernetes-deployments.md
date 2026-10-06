# Phase 1.4 — Kubernetes Deployments

## 1. What is a Deployment?

A Deployment manages a set of Pods and provides controlled application releases.

A Deployment normally manages a ReplicaSet, which manages the actual Pods.

```text
Deployment
     ↓
ReplicaSet
     ↓
Pods
```

Deployments are commonly used for stateless applications.

---

# 2. Why Not Create Pods Directly?

Suppose you create:

```text
Pod A
```

If it crashes or is deleted, there is no higher-level controller responsible for maintaining a desired replica count.

A Deployment gives you:

```text
Desired replicas = 3
```

Kubernetes then maintains approximately:

```text
Pod 1
Pod 2
Pod 3
```

If one disappears:

```text
Pod 1
Pod 2
Pod 3 → deleted
          ↓
       ReplicaSet
          ↓
      New Pod
```

---

# 3. Basic Deployment YAML

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
          image: nginx:1.27
          ports:
            - containerPort: 80
```

The important relationship is:

```yaml
selector:
  matchLabels:
    app: nginx
```

and:

```yaml
template:
  metadata:
    labels:
      app: nginx
```

The selector must correctly identify the Pods created by the Deployment.

---

# 4. Deployment Creation Flow

When you run:

```bash
kubectl apply -f deployment.yaml
```

the simplified flow is:

```text
Deployment
    ↓
ReplicaSet
    ↓
Pods
    ↓
Scheduler
    ↓
Worker Node
    ↓
kubelet
    ↓
Container Runtime
    ↓
Container
```

---

# 5. Replicas

Example:

```yaml
spec:
  replicas: 3
```

This expresses:

> I want three replicas of the application's Pod template.

Check:

```bash
kubectl get deployment
```

Example output conceptually:

```text
NAME    READY   UP-TO-DATE   AVAILABLE
nginx   3/3     3            3
```

---

# 6. Scaling

Scale from 3 to 5:

```bash
kubectl scale deployment nginx --replicas=5
```

Check:

```bash
kubectl get pods
```

You should see additional Pods being created.

Scale down:

```bash
kubectl scale deployment nginx --replicas=2
```

Kubernetes will reduce the number of Pods toward the desired state.

---

# 7. Deployment and ReplicaSet

Suppose the Deployment creates:

```text
Deployment nginx
       ↓
ReplicaSet nginx-abc123
       ↓
Pod 1
Pod 2
Pod 3
```

Now you update the image:

```yaml
image: nginx:1.28
```

Kubernetes creates a new ReplicaSet.

```text
Deployment
   ├── Old ReplicaSet
   │      ├── old Pod
   │      └── old Pod
   │
   └── New ReplicaSet
          ├── new Pod
          └── new Pod
```

The Deployment gradually moves traffic/workload from the old version toward the new version according to its update strategy.

---

# 8. Rolling Update

Rolling updates allow a Deployment to update Pods gradually.

Example:

```text
Version 1
Pod 1
Pod 2
Pod 3
```

Update:

```text
Version 2
```

Kubernetes can gradually replace the old Pods.

Conceptually:

```text
Old Pod → New Pod
Old Pod → New Pod
Old Pod → New Pod
```

This reduces downtime compared with deleting every old Pod at once.

---

# 9. Deployment Strategy

The common Deployment strategy is:

```yaml
strategy:
  type: RollingUpdate
```

The other strategy is:

```yaml
strategy:
  type: Recreate
```

## RollingUpdate

Gradually replace old Pods.

## Recreate

Terminate existing Pods before creating the new set.

Choose based on application requirements.

---

# 10. maxSurge

`maxSurge` controls how many Pods can temporarily exist above the desired replica count during a rolling update.

Example:

```yaml
strategy:
  rollingUpdate:
    maxSurge: 1
```

If desired replicas are:

```text
3
```

Kubernetes can temporarily have more than 3 Pods while updating.

---

# 11. maxUnavailable

`maxUnavailable` controls how many Pods can be unavailable during the update.

Example:

```yaml
strategy:
  rollingUpdate:
    maxUnavailable: 1
```

The combination of `maxSurge` and `maxUnavailable` controls the rollout behavior.

---

# 12. Deployment Revision

Deployments maintain rollout history.

Check:

```bash
kubectl rollout history deployment nginx
```

Example:

```text
deployment.apps/nginx
REVISION  CHANGE-CAUSE
1         Initial deployment
2         Updated image
```

---

# 13. Rollout Status

Check:

```bash
kubectl rollout status deployment nginx
```

This helps determine whether the rollout completed successfully.

---

# 14. Rollback

If version 2 is broken:

```bash
kubectl rollout undo deployment nginx
```

Check:

```bash
kubectl rollout status deployment nginx
```

Then:

```bash
kubectl get pods
```

---

# 15. Pause and Resume

Deployments can be paused.

```bash
kubectl rollout pause deployment nginx
```

Resume:

```bash
kubectl rollout resume deployment nginx
```

This can be useful when making multiple related changes.

---

# 16. Updating an Image

Use:

```bash
kubectl set image deployment/nginx nginx=nginx:1.28
```

Then:

```bash
kubectl rollout status deployment/nginx
```

Inspect:

```bash
kubectl get rs
kubectl get pods
```

Notice the old and new ReplicaSets.

---

# 17. Deployment Status

Useful command:

```bash
kubectl get deployment nginx
```

Common columns include:

```text
READY
UP-TO-DATE
AVAILABLE
```

### READY

Number of ready replicas.

### UP-TO-DATE

Replicas using the current Deployment template.

### AVAILABLE

Replicas available to serve according to Deployment availability rules.

---

# 18. Self-Healing

Suppose:

```text
Desired = 3
Current = 3
```

Delete one Pod:

```bash
kubectl delete pod <pod-name>
```

Now:

```text
Desired = 3
Current = 2
```

ReplicaSet detects the difference and creates another Pod.

```text
3 desired
   ↓
2 existing
   ↓
ReplicaSet
   ↓
Create Pod
   ↓
3 existing
```

This is a practical demonstration of reconciliation.

---

# 19. Deployment YAML Deep Dive

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: web

spec:
  replicas: 3

  selector:
    matchLabels:
      app: web

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0

  template:
    metadata:
      labels:
        app: web

    spec:
      containers:
        - name: web
          image: nginx:1.27
          ports:
            - containerPort: 80
```

### Important hierarchy

```text
Deployment
└── spec
    ├── replicas
    ├── selector
    ├── strategy
    └── template
        ├── metadata
        │   └── labels
        └── spec
            └── containers
```

---

# 20. Selector and Template Labels

This is extremely important.

Deployment:

```yaml
selector:
  matchLabels:
    app: web
```

Pod template:

```yaml
template:
  metadata:
    labels:
      app: web
```

The Deployment uses the selector to identify the Pods it manages.

Incorrect selector/template relationships can prevent the object from behaving as intended.

---

# 21. Deployment Commands

Create:

```bash
kubectl apply -f deployment.yaml
```

List:

```bash
kubectl get deployments
```

Detailed:

```bash
kubectl describe deployment nginx
```

Get ReplicaSets:

```bash
kubectl get rs
```

Get Pods:

```bash
kubectl get pods
```

Scale:

```bash
kubectl scale deployment nginx --replicas=5
```

Set image:

```bash
kubectl set image deployment/nginx nginx=nginx:1.28
```

Rollout status:

```bash
kubectl rollout status deployment/nginx
```

History:

```bash
kubectl rollout history deployment/nginx
```

Undo:

```bash
kubectl rollout undo deployment/nginx
```

---

# 22. Hands-On Exercise — First Deployment

Create:

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
          image: nginx:1.27
```

Apply:

```bash
kubectl apply -f deployment.yaml
```

Check:

```bash
kubectl get deployment
kubectl get rs
kubectl get pods
```

Draw:

```text
Deployment
     ↓
ReplicaSet
     ↓
3 Pods
```

---

# 23. Hands-On Exercise — Scaling

Start with:

```yaml
replicas: 3
```

Then:

```bash
kubectl scale deployment nginx --replicas=5
```

Observe:

```bash
kubectl get pods -w
```

Then scale down:

```bash
kubectl scale deployment nginx --replicas=2
```

---

# 24. Hands-On Exercise — Rolling Update

Change:

```yaml
image: nginx:1.27
```

to:

```yaml
image: nginx:1.28
```

Apply:

```bash
kubectl apply -f deployment.yaml
```

Watch:

```bash
kubectl rollout status deployment/nginx
```

Inspect:

```bash
kubectl get rs
kubectl get pods
```

You should see a new ReplicaSet being used.

---

# 25. Hands-On Exercise — Rollback

Introduce a bad image:

```yaml
image: nginx:does-not-exist
```

Apply it:

```bash
kubectl apply -f deployment.yaml
```

Check:

```bash
kubectl get pods
kubectl describe pod <pod>
```

Then:

```bash
kubectl rollout history deployment/nginx
```

Rollback:

```bash
kubectl rollout undo deployment/nginx
```

Check:

```bash
kubectl rollout status deployment/nginx
```

---

# 26. Hands-On Exercise — Self-Healing

Check Pods:

```bash
kubectl get pods
```

Delete one:

```bash
kubectl delete pod <pod-name>
```

Immediately watch:

```bash
kubectl get pods -w
```

Ask:

> Who recreated the Pod?

Answer:

```text
Deployment
   ↓
ReplicaSet
   ↓
Detects fewer Pods than desired
   ↓
Creates replacement Pod
```

---

# 27. Troubleshooting a Deployment

## Deployment has 0 ready replicas

Check:

```bash
kubectl get deployment
kubectl describe deployment nginx
kubectl get rs
kubectl get pods
```

Then inspect:

```bash
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get events
```

---

## Pods are Pending

Possible causes:

- Insufficient resources
- Taints
- Affinity constraints
- Unsatisfied PVC
- Scheduling restrictions

Check:

```bash
kubectl describe pod <pod>
```

---

## Pods are CrashLoopBackOff

Check:

```bash
kubectl logs <pod>
kubectl logs <pod> --previous
kubectl describe pod <pod>
```

---

## Rollout is stuck

Check:

```bash
kubectl rollout status deployment/nginx
kubectl get pods
kubectl describe deployment nginx
kubectl get events
```

---

# 28. Deployment vs ReplicaSet vs Pod

This is one of the most important relationships in Kubernetes.

```text
Deployment
```

Manages application rollout and ReplicaSets.

```text
ReplicaSet
```

Maintains the desired number of Pods.

```text
Pod
```

Runs the containers.

Think:

```text
Deployment = Release Manager
ReplicaSet = Replica Manager
Pod = Running Workload
```

This is a conceptual analogy, not a literal Kubernetes implementation definition.

---

# 29. Deployment vs StatefulSet

## Deployment

Best for many stateless workloads:

```text
Frontend
API
Web application
Stateless workers
```

Pods are generally interchangeable.

## StatefulSet

Best when Pods need stable identity/storage:

```text
database-0
database-1
database-2
```

Examples:

- Databases
- Distributed systems
- Stateful clustered applications

---

# 30. Deployment vs DaemonSet

## Deployment

You want a specified number of replicas.

```text
3 replicas
```

## DaemonSet

You want a Pod on each eligible node.

```text
Node 1 → Pod
Node 2 → Pod
Node 3 → Pod
```

---

# 31. Interview Questions

1. What is a Deployment?
2. Why use Deployment instead of a Pod?
3. What is the relationship between Deployment and ReplicaSet?
4. What happens when a Pod managed by a Deployment is deleted?
5. How do you scale a Deployment?
6. What is a rolling update?
7. What is `maxSurge`?
8. What is `maxUnavailable`?
9. How do you rollback a Deployment?
10. How do you check rollout status?
11. How do you inspect rollout history?
12. What happens when you change the container image?
13. Why does a new ReplicaSet appear after an update?
14. What causes a rollout to get stuck?
15. Deployment vs StatefulSet?
16. Deployment vs DaemonSet?
17. How does self-healing work?
18. What is the difference between desired replicas and available replicas?

---

# 32. Must Understand

You should be able to explain this without notes:

```text
Deployment
     │
     │ creates/manages
     ▼
ReplicaSet
     │
     │ maintains
     ▼
Pods
     │
     │ run
     ▼
Containers
```

And during an update:

```text
Deployment
   │
   ├── Old ReplicaSet
   │       └── Old Pods
   │
   └── New ReplicaSet
           └── New Pods
```

The Deployment gradually changes the number of old and new replicas according to the rollout strategy.

Once this is clear, Services, Ingress, HPA, Helm, and GitOps become much easier to understand.
