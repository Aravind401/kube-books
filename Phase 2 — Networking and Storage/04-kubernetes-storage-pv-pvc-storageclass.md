# Phase 2.4 — Kubernetes Storage: Volumes, PV, PVC and StorageClass

## 1. Why Storage Matters

Containers are generally treated as ephemeral.

If a container writes data inside its writable container filesystem, that data may disappear when the container is replaced.

For stateful applications, data must live independently from the application container.

```text
Pod
  ↓
Container
  ↓
Ephemeral filesystem
```

For persistent data:

```text
Pod
  ↓
PVC
  ↓
PV
  ↓
Persistent Storage
```

---

# 2. Kubernetes Volumes

A volume provides storage to containers in a Pod.

A volume can have different lifecycles and backends.

Example:

```yaml
volumes:
  - name: shared
    emptyDir: {}
```

Mount:

```yaml
volumeMounts:
  - name: shared
    mountPath: /data
```

---

# 3. emptyDir

`emptyDir` is created when a Pod is assigned to a node.

It exists for the life of the Pod.

```text
Pod created
   ↓
emptyDir created
   ↓
Containers use it
   ↓
Pod deleted
   ↓
emptyDir deleted
```

Useful for:

- Temporary files
- Scratch space
- Sharing files between containers in a Pod

It is not persistent storage for surviving Pod recreation.

---

# 4. hostPath

`hostPath` mounts a path from the node filesystem.

Example:

```yaml
volumes:
  - name: host-data
    hostPath:
      path: /data
```

This tightly couples the workload to node storage.

Use it carefully, especially in production.

It can be useful for certain node-level workloads and local development.

---

# 5. PersistentVolume

A PersistentVolume (PV) represents storage available to the cluster.

Conceptually:

```text
PersistentVolume
       ↓
Physical / Cloud Storage
```

Examples of underlying storage can include cloud disks, network filesystems, or CSI-backed storage.

---

# 6. PersistentVolumeClaim

A PersistentVolumeClaim (PVC) is a request for storage by a workload.

Example:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: app-data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

Think:

```text
PV = Storage resource
PVC = Request for storage
```

---

# 7. StorageClass

StorageClass defines how storage can be dynamically provisioned.

Example concept:

```text
PVC
 ↓
StorageClass
 ↓
CSI Driver
 ↓
Cloud Storage
 ↓
PV
```

A cluster may have different StorageClasses:

```text
fast-ssd
standard
encrypted
```

Inspect:

```bash
kubectl get storageclass
```

---

# 8. Dynamic Provisioning

Instead of manually creating a PV:

```text
Developer creates PVC
        ↓
StorageClass
        ↓
Provisioner / CSI driver
        ↓
Storage automatically created
        ↓
PV
        ↓
PVC bound
```

This is the common pattern in cloud Kubernetes environments.

---

# 9. CSI

CSI means:

> Container Storage Interface

CSI provides a standard interface for storage plugins.

Conceptually:

```text
Kubernetes
    ↓
CSI
    ↓
Storage Driver
    ↓
Cloud / SAN / NAS / Other Storage
```

Examples include cloud-specific CSI drivers.

---

# 10. Access Modes

Common access modes include:

### ReadWriteOnce (RWO)

The volume can be mounted read-write by a single node, subject to the storage implementation.

### ReadOnlyMany (ROX)

The volume can be mounted read-only by multiple nodes, if supported.

### ReadWriteMany (RWX)

The volume can be mounted read-write by multiple nodes, if supported.

### ReadWriteOncePod (RWOP)

Restricts read-write mounting to a single Pod, when supported by the storage stack.

Always verify which modes the actual storage driver supports.

---

# 11. PVC Example

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: database-data
spec:
  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 20Gi

  storageClassName: standard
```

Check:

```bash
kubectl get pvc
```

---

# 12. Mount PVC into a Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: storage-demo
spec:
  containers:
    - name: app
      image: nginx:1.27
      volumeMounts:
        - name: data
          mountPath: /data

  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: database-data
```

Flow:

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

# 13. PV/PVC Lifecycle

Conceptually:

```text
Available
   ↓
Bound
   ↓
Released
   ↓
Reclaimed
```

The exact lifecycle and reclaim behavior depend on the PV and StorageClass configuration.

Inspect:

```bash
kubectl get pv
kubectl get pvc
```

---

# 14. Reclaim Policy

Common PV reclaim policies include:

- Retain
- Delete

## Retain

The storage resource is retained after the PVC is released, requiring administrator action depending on the storage backend.

## Delete

The associated storage resource may be deleted when the PVC is deleted, depending on the provisioner and configuration.

Be very careful with production databases.

---

# 15. StatefulSet and Storage

StatefulSets commonly use `volumeClaimTemplates`.

Example concept:

```text
StatefulSet
   ├── database-0
   │      └── PVC
   │
   ├── database-1
   │      └── PVC
   │
   └── database-2
          └── PVC
```

This provides stable storage association with Pod identities.

---

# 16. Headless Service + StatefulSet

A common stateful architecture:

```text
                Headless Service
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
      database-0   database-1   database-2
          │            │            │
         PVC          PVC          PVC
```

This is useful for distributed databases and clustered applications.

---

# 17. StorageClass Example

The exact fields depend on the CSI driver.

Generic structure:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-storage
provisioner: example.com/csi-driver
parameters:
  type: fast
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

Do not copy the example provisioner into a real cluster unless that driver exists.

---

# 18. volumeBindingMode

A StorageClass can use:

```yaml
volumeBindingMode: WaitForFirstConsumer
```

This can delay volume provisioning/binding until a workload using the PVC is scheduled.

This is especially useful when storage has topology constraints.

---

# 19. Storage Topology

Cloud disks can be tied to:

- Availability zones
- Regions
- Nodes
- Storage systems

Kubernetes and CSI drivers use topology information to place workloads and storage correctly.

Example:

```text
Zone A
 └── Disk A

Zone B
 └── Disk B
```

A Pod may need to be scheduled where its volume can be attached.

---

# 20. Troubleshooting PVCs

## PVC Pending

Check:

```bash
kubectl get pvc
kubectl describe pvc <pvc>
kubectl get storageclass
kubectl get pv
```

Possible causes:

- No matching StorageClass
- Storage provisioner unavailable
- Unsupported access mode
- Insufficient capacity
- Topology issue
- CSI driver problem

---

# 21. Troubleshooting Mount Problems

Check:

```bash
kubectl describe pod <pod>
kubectl get pvc
kubectl get pv
kubectl get events
```

Look for:

- Attach failures
- Mount failures
- Permission problems
- Volume topology problems
- CSI errors

---

# 22. Storage in Minikube

For local learning, Minikube may provide a default StorageClass/provisioner depending on its configuration.

Inspect:

```bash
kubectl get storageclass
kubectl get pv
kubectl get pvc
```

Do not assume local Minikube storage behaves like EKS, AKS, or GKE storage.

---

# 23. Database Example

Conceptually:

```text
Application Deployment
        ↓
Database Service
        ↓
Database StatefulSet
        ↓
PVC
        ↓
PV
        ↓
Persistent Storage
```

For production databases, also consider managed database services rather than automatically running every database inside Kubernetes.

---

# 24. Hands-On Lab — PVC

Create:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Apply:

```bash
kubectl apply -f pvc.yaml
```

Check:

```bash
kubectl get pvc
kubectl get pv
```

---

# 25. Hands-On Lab — Mount Storage

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: storage-demo
spec:
  containers:
    - name: app
      image: busybox:1.36
      command:
        - sh
        - -c
        - |
          echo "hello kubernetes" > /data/message.txt
          sleep 3600

      volumeMounts:
        - name: data
          mountPath: /data

  volumes:
    - name: data
      persistentVolumeClaim:
        claimName: demo-pvc
```

Apply:

```bash
kubectl apply -f storage-demo.yaml
```

Check:

```bash
kubectl exec storage-demo -- cat /data/message.txt
```

---

# 26. Storage Persistence Exercise

Delete the Pod:

```bash
kubectl delete pod storage-demo
```

Create the Pod again using the same PVC.

Check:

```bash
kubectl exec storage-demo -- cat /data/message.txt
```

The goal is to understand:

```text
Pod deleted
   ↓
PVC remains
   ↓
Storage remains
   ↓
New Pod mounts same storage
```

The exact persistence behavior depends on the storage backend and reclaim/configuration settings.

---

# 27. Interview Questions

1. Why do containers need persistent storage?
2. What is a Kubernetes Volume?
3. What is `emptyDir`?
4. What is `hostPath`?
5. What is a PersistentVolume?
6. What is a PersistentVolumeClaim?
7. PV vs PVC?
8. What is a StorageClass?
9. What is dynamic provisioning?
10. What is CSI?
11. What are access modes?
12. What is RWO?
13. What is RWX?
14. What is RWOP?
15. What is a reclaim policy?
16. What is `WaitForFirstConsumer`?
17. Why can a PVC remain Pending?
18. How does StatefulSet use persistent storage?
19. What is storage topology?
20. How would you troubleshoot a volume mount failure?

---

# 28. Must Understand

Memorize this relationship:

```text
Application
    ↓
Pod
    ↓
PVC
    ↓
PV
    ↓
StorageClass / CSI
    ↓
Actual Storage
```

And:

```text
PVC = Application's request
PV  = Storage resource
CSI = Storage integration
StorageClass = Dynamic provisioning policy
```

These four concepts are foundational for Kubernetes storage.
