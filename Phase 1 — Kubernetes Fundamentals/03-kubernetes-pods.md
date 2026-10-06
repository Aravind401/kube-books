# Phase 1.3 — Kubernetes Pods

## 1. What is a Pod?

A Pod is the smallest deployable unit in Kubernetes.

A Pod represents one or more containers that are scheduled together on the same node.

```text
Pod
├── Container A
├── Container B
└── Shared resources
```

Containers in the same Pod share:

- Network namespace
- Pod IP
- Local volumes
- Lifecycle

---

# 2. Why Does Kubernetes Use Pods?

Kubernetes could theoretically manage individual containers, but Pods provide a useful abstraction for workloads that need to run together.

For example:

```text
Pod
├── Application container
└── Sidecar container
```

Both containers can communicate through:

```text
localhost
```

because they share the Pod network namespace.

---

# 3. One Container vs Multiple Containers

## Single-container Pod

Most applications use one main container per Pod.

```text
Pod
└── Application
```

## Multi-container Pod

Sometimes tightly coupled containers belong together.

```text
Pod
├── Application
└── Sidecar
```

Examples can include:

- Proxy sidecar
- Log helper
- Security helper
- Adapter container

Do not put unrelated applications into one Pod simply because they need to run on Kubernetes.

---

# 4. Pod YAML

Basic example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.27
      ports:
        - containerPort: 80
```

Create:

```bash
kubectl apply -f pod.yaml
```

Check:

```bash
kubectl get pod nginx
```

---

# 5. Pod Lifecycle

A simplified Pod lifecycle is:

```text
Pending
   ↓
Running
   ↓
Succeeded / Failed
```

Pods may also be observed through statuses such as:

```text
ContainerCreating
CrashLoopBackOff
Terminating
```

These are not all Pod phases; some are displayed container or waiting states.

---

# 6. Pod Phases

The primary Pod phases are:

- Pending
- Running
- Succeeded
- Failed
- Unknown

### Pending

The Pod has been accepted but cannot yet run.

Possible causes:

- Scheduling problem
- Insufficient resources
- Image pulling
- Volume setup
- Scheduling constraints

### Running

At least one container is running or starting.

### Succeeded

All containers terminated successfully.

### Failed

All containers terminated and at least one failed.

### Unknown

The cluster cannot determine the Pod state.

---

# 7. Pod Conditions

Pod conditions provide more detailed status information.

Common conditions include:

- PodScheduled
- Initialized
- ContainersReady
- Ready

Check:

```bash
kubectl describe pod <pod>
```

---

# 8. Pod IP

A Pod normally receives an IP address from the cluster network.

Example:

```text
Pod A → 10.x.x.10
Pod B → 10.x.x.11
```

Pod IPs are generally ephemeral.

If a Pod is deleted and recreated, its IP may change.

That is one reason Services are important.

```text
Client
  ↓
Service
  ↓
Current Pod
```

---

# 9. Pod Networking

Containers in the same Pod share the network namespace.

Example:

```text
Pod
 ├── App :8080
 └── Sidecar :9000
```

The application container could communicate with the sidecar using:

```text
localhost:9000
```

Pods on different nodes communicate through the cluster's networking implementation.

---

# 10. Pod Volumes

Containers in a Pod can share mounted volumes.

Example:

```yaml
spec:
  volumes:
    - name: shared
      emptyDir: {}

  containers:
    - name: app
      image: nginx
      volumeMounts:
        - name: shared
          mountPath: /data

    - name: helper
      image: busybox
      volumeMounts:
        - name: shared
          mountPath: /data
```

Both containers can access `/data`.

---

# 11. Init Containers

Init containers run before application containers.

Example:

```yaml
spec:
  initContainers:
    - name: init
      image: busybox
      command:
        - sh
        - -c
        - echo "Initializing..."

  containers:
    - name: app
      image: nginx
```

Flow:

```text
Init Container
      ↓
Completed successfully
      ↓
Application Container
```

Common uses:

- Initialization
- Preparing files
- Waiting for prerequisites
- Setup tasks

---

# 12. Sidecar Containers

A sidecar runs alongside the main application.

```text
Pod
├── Main Application
└── Sidecar
```

The sidecar can provide supporting functionality.

Examples:

- Proxy
- Adapter
- Log processing
- Security helper

---

# 13. Container States

A container can have states such as:

- Waiting
- Running
- Terminated

Inspect:

```bash
kubectl get pod <pod> -o yaml
```

or:

```bash
kubectl describe pod <pod>
```

---

# 14. Restart Policy

Pod restart policies include:

- Always
- OnFailure
- Never

For controllers such as Deployments, Pods normally use:

```yaml
restartPolicy: Always
```

---

# 15. Liveness Probe

A liveness probe answers:

> Is this container still healthy enough to keep running?

If liveness repeatedly fails, kubelet may restart the container.

Example:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 10
```

---

# 16. Readiness Probe

Readiness answers:

> Is this Pod ready to receive traffic?

A Pod can be running but not ready.

```text
Pod Running
     ↓
Readiness = false
     ↓
Service does not send normal traffic to it
```

Example:

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  periodSeconds: 5
```

---

# 17. Startup Probe

Startup probes help applications that take a long time to initialize.

```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

The startup probe can prevent liveness checking from restarting an application while it is still starting.

---

# 18. Requests and Limits

Example:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

### Request

The amount of resource Kubernetes uses when making scheduling decisions.

### Limit

The maximum resource allocation allowed by the configured resource controls.

Memory limits are especially important because exceeding a memory limit can result in termination/OOM behavior.

---

# 19. QoS Classes

Kubernetes assigns QoS classes based on resource configuration.

### Guaranteed

Strictly configured CPU and memory requests/limits for containers.

### Burstable

Some resource requests/limits exist but do not meet Guaranteed criteria.

### BestEffort

No CPU or memory requests/limits are specified.

Understanding QoS is useful when analyzing resource pressure and eviction behavior.

---

# 20. Pod Eviction

Under resource pressure, Kubernetes may evict Pods.

Example:

```text
Node memory pressure
        ↓
Kubernetes evaluates Pods
        ↓
Lower-priority / lower-QoS workloads may be affected
        ↓
Pod eviction
```

The exact behavior depends on node conditions, QoS, priority, and other policies.

---

# 21. Pod Security Context

Security settings can be applied to Pods and containers.

Example:

```yaml
spec:
  securityContext:
    runAsNonRoot: true
```

Container-level settings can include:

- runAsUser
- runAsGroup
- allowPrivilegeEscalation
- capabilities
- readOnlyRootFilesystem

---

# 22. Pod Commands

Create:

```bash
kubectl apply -f pod.yaml
```

List:

```bash
kubectl get pods
```

Detailed information:

```bash
kubectl describe pod nginx
```

Logs:

```bash
kubectl logs nginx
```

Previous container logs:

```bash
kubectl logs nginx --previous
```

Execute a command:

```bash
kubectl exec -it nginx -- /bin/sh
```

View YAML:

```bash
kubectl get pod nginx -o yaml
```

Delete:

```bash
kubectl delete pod nginx
```

---

# 23. Troubleshooting Pods

## Pending

Check:

```bash
kubectl describe pod <pod>
kubectl get events
```

Look for:

- Insufficient CPU
- Insufficient memory
- Taints
- Affinity rules
- PVC problems

## CrashLoopBackOff

Check:

```bash
kubectl logs <pod>
kubectl logs <pod> --previous
kubectl describe pod <pod>
```

Possible causes:

- Application crash
- Incorrect configuration
- Missing environment variables
- Dependency unavailable
- Bad startup command

## ImagePullBackOff

Check:

```bash
kubectl describe pod <pod>
```

Possible causes:

- Image does not exist
- Wrong tag
- Private registry authentication
- Network problems

---

# 24. Hands-On Exercise

Create:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web
spec:
  containers:
    - name: nginx
      image: nginx:1.27
      ports:
        - containerPort: 80
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "500m"
          memory: "256Mi"
```

Apply:

```bash
kubectl apply -f pod.yaml
```

Inspect:

```bash
kubectl get pod web
kubectl describe pod web
kubectl logs web
```

Enter the container:

```bash
kubectl exec -it web -- /bin/sh
```

---

# 25. Failure Exercise

Change:

```yaml
image: nginx:does-not-exist
```

Apply again.

Then:

```bash
kubectl describe pod web
```

Find the image-pull error.

Restore the correct image and observe the Pod recover.

---

# 26. Important Interview Questions

1. What is a Pod?
2. Why does Kubernetes use Pods?
3. Can a Pod contain multiple containers?
4. Do containers in a Pod share an IP?
5. Do containers in a Pod share volumes?
6. What is a Pod IP?
7. Why are Pod IPs considered ephemeral?
8. What is an init container?
9. What is a sidecar?
10. What is a liveness probe?
11. What is a readiness probe?
12. What is a startup probe?
13. Liveness vs readiness?
14. What is `CrashLoopBackOff`?
15. Why does a Pod remain Pending?
16. What are CPU/memory requests?
17. What are CPU/memory limits?
18. What are Kubernetes QoS classes?
19. What happens when a Pod is deleted?
20. Why should applications normally be managed through controllers?

---

# 27. Must Understand

Be able to explain:

```text
Pod
├── Container
├── Network namespace
├── Pod IP
├── Volumes
└── Lifecycle
```

And:

```text
Running ≠ Ready
```

A container can be running while the application is not ready to receive traffic.

That distinction becomes extremely important when you learn Services, Deployments, and production troubleshooting.
