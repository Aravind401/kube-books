# Phase 2.1 — Kubernetes Services and Networking

## 1. Why Kubernetes Networking Matters

Pods are ephemeral.

A Pod can be:

- Created
- Deleted
- Recreated
- Moved to another node
- Given a different IP address

Therefore, applications should not normally depend directly on a Pod IP.

Kubernetes networking provides a way for workloads to communicate reliably.

A simplified model is:

```text
Client
   ↓
Service
   ↓
Pod
```

---

# 2. Kubernetes Networking Model

A Kubernetes cluster normally provides networking where:

1. Every Pod gets its own IP address.
2. Pods can communicate with other Pods according to the cluster network implementation.
3. Nodes can communicate with Pods.
4. Pods can communicate with Services.
5. Services provide stable virtual endpoints.

Conceptually:

```text
             Kubernetes Cluster
                    │
       ┌────────────┴────────────┐
       │                         │
    Node 1                    Node 2
       │                         │
    Pod A ─────────────────── Pod B
       │                         │
       └────────── Service ──────┘
```

The exact implementation depends on the cluster's CNI/networking solution.

---

# 3. Pod IP

When a Pod is created, the cluster networking layer assigns it an IP.

Example:

```text
frontend Pod → 10.244.1.10
backend Pod  → 10.244.2.15
```

Pod IPs are not normally stable application endpoints.

If the backend Pod is recreated:

```text
Old backend Pod
10.244.2.15
     ↓
Deleted

New backend Pod
10.244.1.22
```

The application should use a Service instead.

---

# 4. What is a Service?

A Service provides a stable network abstraction over a set of Pods.

Example:

```text
              backend-service
                    │
              selector:
             app=backend
                    │
          ┌─────────┼─────────┐
          ↓         ↓         ↓
       Pod 1      Pod 2      Pod 3
```

The Pods may change, but the Service remains the stable endpoint.

---

# 5. Service Selectors

Example Pod:

```yaml
metadata:
  labels:
    app: backend
```

Service:

```yaml
spec:
  selector:
    app: backend
```

The Service finds matching Pods using labels.

This relationship is extremely important:

```text
Service selector
       ↓
Pod labels
       ↓
Matching Pods
```

---

# 6. Basic ClusterIP Service

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

Meaning:

```text
Service port: 80
        ↓
Pod port: 8080
```

---

# 7. Service Port vs TargetPort

Consider:

```yaml
ports:
  - port: 80
    targetPort: 8080
```

`port` is the Service port.

`targetPort` is the destination port on the selected Pod.

```text
Client
  ↓
Service :80
  ↓
Pod :8080
```

They do not have to be the same number.

---

# 8. ClusterIP

ClusterIP is the default Service type.

It provides an internal virtual IP accessible inside the cluster.

```text
Pod A
  ↓
backend-service
  ↓
Backend Pods
```

Use it for:

- Internal APIs
- Backend services
- Databases
- Internal application components

---

# 9. NodePort

NodePort exposes a Service on a port on each eligible node.

Conceptually:

```text
External Client
      ↓
NodeIP:NodePort
      ↓
Service
      ↓
Pod
```

Example:

```yaml
type: NodePort
```

A NodePort is useful for simple external access, development, and certain infrastructure setups, but production applications often use a cloud LoadBalancer or Ingress instead.

---

# 10. LoadBalancer

LoadBalancer asks the underlying environment to provide an external load balancer when supported.

Conceptually:

```text
Internet
   ↓
Cloud Load Balancer
   ↓
Service
   ↓
Pods
```

Common with managed Kubernetes platforms such as:

- EKS
- AKS
- GKE

---

# 11. ExternalName

ExternalName can provide a DNS alias to an external hostname.

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-db
spec:
  type: ExternalName
  externalName: database.example.com
```

It does not create a normal ClusterIP-backed Service.

---

# 12. Service Types Summary

| Type | Main Use |
|---|---|
| ClusterIP | Internal cluster communication |
| NodePort | Expose through node ports |
| LoadBalancer | External load balancer |
| ExternalName | DNS alias to an external service |

---

# 13. Service Discovery

Kubernetes provides service discovery through DNS.

Suppose:

```text
Service:
backend
Namespace:
production
```

A workload can usually resolve:

```text
backend
```

within the appropriate DNS search context.

A fully qualified Kubernetes Service DNS name follows the pattern:

```text
<service>.<namespace>.svc.<cluster-domain>
```

For example:

```text
backend.production.svc.cluster.local
```

The exact cluster domain may differ from `cluster.local`.

---

# 14. CoreDNS

CoreDNS provides DNS-based service discovery in many Kubernetes installations.

Conceptually:

```text
Application
    ↓
DNS lookup
    ↓
CoreDNS
    ↓
Service DNS
    ↓
Service IP
```

Check CoreDNS:

```bash
kubectl get pods -n kube-system
```

Depending on the distribution, CoreDNS Pods may have names beginning with:

```text
coredns-
```

---

# 15. EndpointSlices

A Service needs to know which endpoints correspond to its selector.

Modern Kubernetes uses EndpointSlices for scalable endpoint tracking.

Conceptually:

```text
Service
   ↓
EndpointSlices
   ↓
Pod endpoints
```

Inspect:

```bash
kubectl get endpointslices
```

---

# 16. What Happens When a Pod Is Recreated?

Suppose:

```text
Service
   ↓
Pod A
Pod B
Pod C
```

Pod B is deleted.

```text
Service
   ↓
Pod A
Pod C
```

A replacement Pod appears:

```text
Service
   ↓
Pod A
Pod C
Pod D
```

The Service's endpoint information is updated.

Applications do not need to know the new Pod IP.

---

# 17. Service Load Distribution

A Service can distribute connections among matching endpoints.

The exact traffic distribution depends on the Service implementation, kube-proxy/dataplane, connection behavior, and cluster networking.

Do not think of a Service as a full-featured application-layer load balancer.

It provides stable service networking.

---

# 18. Headless Service

A headless Service uses:

```yaml
clusterIP: None
```

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: database
spec:
  clusterIP: None
  selector:
    app: database
  ports:
    - port: 5432
```

Headless Services are commonly used with StatefulSets.

They allow DNS to expose individual Pod identities.

---

# 19. Service Without Selector

A Service can also be created without a selector and used with manually managed EndpointSlices.

This can be useful for advanced integrations and external services.

This is an advanced topic and can be studied after normal selector-based Services.

---

# 20. Network Policies

NetworkPolicy controls which traffic is allowed to or from Pods when the cluster networking implementation supports NetworkPolicy.

Example architecture:

```text
Frontend ───────► Backend       ✅
Backend ────────► Database      ✅
Frontend ───────► Database      ❌
Internet ───────► Database      ❌
```

NetworkPolicy can control:

- Ingress
- Egress
- Pod selectors
- Namespace selectors
- IP blocks

NetworkPolicy behavior depends on the installed CNI supporting it.

---

# 21. Simple NetworkPolicy Example

Allow only backend Pods to reach the database:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: database-policy
spec:
  podSelector:
    matchLabels:
      app: database

  policyTypes:
    - Ingress

  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: backend
```

The database Pods are selected by:

```yaml
app: database
```

Only Pods labeled:

```yaml
app: backend
```

are allowed by this rule.

---

# 22. CNI

CNI means:

> Container Network Interface

CNI plugins implement Pod networking.

Examples include:

- Cilium
- Calico
- Flannel
- Cloud-provider-specific networking solutions

Conceptually:

```text
Kubernetes
    ↓
CNI
    ↓
Pod network
```

---

# 23. Troubleshooting Services

## Service exists but application is unreachable

Check:

```bash
kubectl get svc
kubectl describe svc <service>
kubectl get endpointslices
kubectl get pods --show-labels
```

Look for selector mismatch.

Example:

Service:

```yaml
selector:
  app: backend
```

Pod:

```yaml
labels:
  app: api
```

These do not match.

Result:

```text
Service
   ↓
No matching endpoints
```

---

# 24. Hands-On Lab — Create a Backend Service

Create a Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
```

Create Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 80
```

Apply:

```bash
kubectl apply -f backend.yaml
```

Check:

```bash
kubectl get svc
kubectl get endpointslices
kubectl get pods --show-labels
```

---

# 25. Hands-On Lab — Test Service DNS

Run a temporary Pod:

```bash
kubectl run dns-test \
  --image=busybox:1.36 \
  --restart=Never \
  --rm -it \
  -- sh
```

Inside the Pod:

```sh
nslookup backend
```

You can also inspect:

```sh
wget -qO- http://backend
```

depending on the test image and application.

---

# 26. Interview Questions

1. What is a Kubernetes Service?
2. Why do we need Services?
3. What is ClusterIP?
4. ClusterIP vs NodePort?
5. NodePort vs LoadBalancer?
6. What is `port`?
7. What is `targetPort`?
8. How does a Service find Pods?
9. What are EndpointSlices?
10. What is CoreDNS?
11. What is a headless Service?
12. What happens when a Service has no endpoints?
13. What is CNI?
14. What is NetworkPolicy?
15. Does every Kubernetes cluster enforce NetworkPolicy automatically?
16. How would you troubleshoot a Service that cannot reach Pods?

---

# 27. Must Understand

Draw this:

```text
Client
   ↓
Service
   ↓
Selector
   ↓
EndpointSlices
   ↓
Pods
```

And:

```text
Service
  │
  ├── Pod A
  ├── Pod B
  └── Pod C

Pod B deleted
  ↓
Endpoint information changes
  ↓
Replacement Pod
  ↓
Service continues to provide stable access
```
