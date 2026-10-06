# Phase 2.2 — Kubernetes DNS and Ingress

## 1. Why DNS?

Pod IP addresses can change.

Services provide stable endpoints.

DNS makes Services easy to discover by name.

Instead of:

```text
http://10.96.20.15
```

an application can use:

```text
http://backend
```

or:

```text
http://backend.production.svc.cluster.local
```

---

# 2. Kubernetes DNS

A typical Kubernetes cluster has a DNS service, commonly CoreDNS.

```text
Application
     ↓
DNS query
     ↓
CoreDNS
     ↓
Service DNS record
     ↓
Service
```

---

# 3. Service DNS Names

The general format is:

```text
<service>.<namespace>.svc.<cluster-domain>
```

Example:

```text
backend.production.svc.cluster.local
```

If the application is in the same namespace, it can often use:

```text
backend
```

The cluster domain can be configured differently, so do not assume it must always be `cluster.local`.

---

# 4. Pod DNS

Pods also have DNS records in certain configurations.

StatefulSets and headless Services are especially important for stable Pod-specific DNS identities.

Example:

```text
database-0.database.production.svc.cluster.local
database-1.database.production.svc.cluster.local
database-2.database.production.svc.cluster.local
```

This pattern is common in stateful applications.

---

# 5. CoreDNS

CoreDNS is commonly deployed in the cluster system namespace.

Inspect:

```bash
kubectl get pods -n kube-system
```

Find CoreDNS:

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

The exact labels can vary by Kubernetes distribution.

---

# 6. DNS Troubleshooting

Create a test Pod:

```bash
kubectl run dns-test \
  --image=busybox:1.36 \
  --restart=Never \
  --rm -it \
  -- sh
```

Then:

```sh
nslookup kubernetes.default
```

Test an application Service:

```sh
nslookup backend
```

Check DNS configuration:

```sh
cat /etc/resolv.conf
```

---

# 7. Common DNS Problems

Possible causes:

- CoreDNS unavailable
- Wrong Service name
- Wrong namespace
- Service has no endpoints
- NetworkPolicy blocks traffic
- Cluster DNS configuration problem
- Application DNS configuration problem

Useful commands:

```bash
kubectl get pods -n kube-system
kubectl get svc -n kube-system
kubectl logs -n kube-system <coredns-pod>
kubectl get svc
kubectl get endpointslices
```

---

# 8. What is Ingress?

Ingress provides HTTP/HTTPS routing into Kubernetes Services.

A common architecture is:

```text
Internet
   ↓
Ingress Controller
   ↓
Ingress Rules
   ↓
Service
   ↓
Pods
```

Ingress is primarily an HTTP/HTTPS routing abstraction.

---

# 9. Ingress vs Service

A Service provides stable networking to Pods.

Ingress provides HTTP/HTTPS routing to Services.

```text
Internet
   ↓
Ingress
   ↓
Service
   ↓
Pod
```

---

# 10. Ingress Controller

An Ingress resource by itself does not automatically route traffic.

A controller must implement it.

Examples:

- NGINX Ingress Controller
- HAProxy-based controllers
- Traefik
- Cloud-provider ingress/load-balancing controllers
- Gateway API implementations

Always distinguish:

```text
Ingress Resource
```

from:

```text
Ingress Controller
```

---

# 11. Basic Ingress Example

Assume a Service exists:

```text
frontend-service
```

Ingress:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: frontend
spec:
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

Traffic:

```text
https://app.example.com
        ↓
Ingress Controller
        ↓
frontend-service
        ↓
Frontend Pods
```

---

# 12. Host-Based Routing

You can route different hostnames:

```text
app.example.com
        ↓
frontend-service

api.example.com
        ↓
backend-service
```

Example:

```yaml
rules:
  - host: app.example.com
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: frontend
              port:
                number: 80

  - host: api.example.com
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: backend
              port:
                number: 80
```

---

# 13. Path-Based Routing

You can route based on URL paths:

```text
example.com/
      ↓
frontend

example.com/api
      ↓
backend
```

Example:

```yaml
rules:
  - host: example.com
    http:
      paths:
        - path: /api
          pathType: Prefix
          backend:
            service:
              name: backend
              port:
                number: 80

        - path: /
          pathType: Prefix
          backend:
            service:
              name: frontend
              port:
                number: 80
```

---

# 14. Path Types

Common path types include:

- Prefix
- Exact

Example:

```yaml
pathType: Prefix
```

means the route matches the path prefix according to Kubernetes Ingress path matching rules.

---

# 15. TLS

Ingress can terminate TLS.

Example:

```yaml
spec:
  tls:
    - hosts:
        - app.example.com
      secretName: app-tls
```

Then:

```text
Client HTTPS
     ↓
Ingress Controller
     ↓
TLS termination
     ↓
Service
     ↓
Pod
```

---

# 16. TLS Secret

A typical TLS Secret contains:

- Certificate
- Private key

Example creation:

```bash
kubectl create secret tls app-tls \
  --cert=tls.crt \
  --key=tls.key
```

Do not commit private keys to Git.

---

# 17. Ingress Annotations

Some Ingress controllers use annotations for controller-specific configuration.

Example:

```yaml
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
```

Annotations are controller-specific and should be checked against the controller documentation/version you are using.

---

# 18. IngressClass

IngressClass identifies which controller should implement an Ingress.

Example:

```yaml
spec:
  ingressClassName: nginx
```

Inspect:

```bash
kubectl get ingressclass
```

---

# 19. Ingress Troubleshooting

Check:

```bash
kubectl get ingress
kubectl describe ingress <name>
kubectl get ingressclass
```

Then:

```bash
kubectl get svc
kubectl get endpointslices
kubectl get pods
```

If using an NGINX controller:

```bash
kubectl get pods -A | grep ingress
```

Check controller logs:

```bash
kubectl logs <ingress-controller-pod>
```

---

# 20. Common Ingress Problems

### 404

Possible causes:

- Wrong path
- Wrong Service
- Wrong Ingress rule
- Application itself returns 404

### 502 / 503

Possible causes:

- Service has no healthy endpoints
- Wrong target port
- Backend application not ready
- Controller cannot reach the Service

### TLS error

Possible causes:

- Wrong Secret
- Invalid certificate
- Host mismatch
- Expired certificate

---

# 21. Ingress vs Gateway API

Ingress remains widely used, but Kubernetes networking also includes the Gateway API project.

Conceptually:

```text
Ingress
   ↓
Simple HTTP/HTTPS routing

Gateway API
   ↓
More expressive traffic management model
```

Learn Ingress first, then study Gateway API as an advanced networking topic.

---

# 22. Hands-On Lab

Create:

```text
Deployment
   ↓
Service
   ↓
Ingress
```

Verify each layer separately.

### Step 1

```bash
kubectl get deployment
```

### Step 2

```bash
kubectl get pods
```

### Step 3

```bash
kubectl get svc
```

### Step 4

```bash
kubectl get ingress
```

### Step 5

Test the Service directly before troubleshooting the Ingress.

This isolates the problem.

---

# 23. Interview Questions

1. What is CoreDNS?
2. How does Service DNS work?
3. What is a fully qualified Kubernetes Service name?
4. What is a headless Service?
5. What is Ingress?
6. Is an Ingress resource itself a proxy?
7. What is an Ingress Controller?
8. Ingress vs Service?
9. Host-based vs path-based routing?
10. How does TLS termination work?
11. What is IngressClass?
12. How would you troubleshoot a 503 from Ingress?
13. What is Gateway API?
14. Why should you test Service connectivity before debugging Ingress?

---

# 24. Must Understand

You should be able to draw:

```text
Internet
   ↓
DNS
   ↓
Load Balancer
   ↓
Ingress Controller
   ↓
Ingress Rules
   ↓
Service
   ↓
EndpointSlices
   ↓
Pods
```

And inside the cluster:

```text
Application
    ↓
Service DNS
    ↓
CoreDNS
    ↓
Service
    ↓
Pods
```
