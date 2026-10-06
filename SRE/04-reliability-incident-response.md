# Kubernetes Phase 2.5 --- Reliability and Incident Response

## 1. Reliability Engineering

Reliability means a system consistently performs its intended function
under expected conditions.

In production Kubernetes, reliability includes:

-   Availability
-   Performance
-   Resilience
-   Recoverability
-   Scalability
-   Security
-   Operability

## 2. Failure Is Expected

Production systems fail.

Examples:

``` text
Pod crashes
Node fails
Database becomes slow
Network partition occurs
Certificate expires
Container image is unavailable
Dependency becomes unavailable
Bad deployment reaches production
```

The goal is not to assume failure never happens.

The goal is to design for failure and recover quickly.

## 3. MTTD and MTTR

### MTTD

Mean Time To Detect.

``` text
Failure occurs
     |
     v
Alert/Detection
```

The time between failure and detection.

### MTTR

Mean Time To Recovery/Repair.

``` text
Failure
  |
Detection
  |
Diagnosis
  |
Mitigation
  |
Recovery
```

Reducing MTTR is a major reliability goal.

## 4. Incident Lifecycle

``` text
Detection
   |
   v
Triage
   |
   v
Mitigation
   |
   v
Root cause investigation
   |
   v
Recovery
   |
   v
Postmortem
   |
   v
Corrective actions
```

## 5. Triage

During an incident ask:

1.  What is broken?
2.  Who is affected?
3.  When did it start?
4.  What changed?
5.  Is the issue getting worse?
6.  Can we mitigate quickly?
7.  What telemetry supports the hypothesis?

## 6. Kubernetes Incident Example

### Symptom

Users receive HTTP 503.

Start with:

``` bash
kubectl get pods -A
kubectl get svc -A
kubectl get ingress -A
kubectl get endpoints -A
kubectl get endpointslices -A
```

Check:

``` bash
kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs <pod> --previous
```

Then inspect:

``` bash
kubectl describe svc <service>
kubectl describe ingress <ingress>
```

## 7. Rollout Incident

Check:

``` bash
kubectl rollout status deployment/<deployment>
kubectl rollout history deployment/<deployment>
kubectl get rs
```

If a deployment is unhealthy:

``` bash
kubectl rollout undo deployment/<deployment>
```

Then investigate the failed release.

Do not treat rollback as the root-cause fix.

## 8. Pod CrashLoopBackOff

Typical investigation:

``` bash
kubectl get pod <pod>
kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs <pod> --previous
```

Look for:

-   Application crash
-   Bad environment variable
-   Missing Secret
-   Missing ConfigMap
-   Failed dependency
-   Incorrect command
-   Failed liveness probe
-   Out-of-memory kill

## 9. OOMKilled

Check:

``` bash
kubectl describe pod <pod>
kubectl top pod <pod>
```

Investigate:

-   Memory request
-   Memory limit
-   Application memory growth
-   JVM/.NET/Node runtime settings
-   Traffic changes

A simple example:

``` yaml
resources:
  requests:
    memory: "256Mi"
  limits:
    memory: "512Mi"
```

Requests and limits must be chosen based on real workload behavior.

## 10. Pod Disruption Budget

PDBs help limit voluntary disruptions.

Example:

``` yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: orders-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: orders
```

PDBs do not protect against every failure.

They primarily control voluntary disruptions such as some node
maintenance/drain scenarios.

## 11. High Availability

For critical services consider:

-   Multiple replicas
-   Multiple nodes
-   Pod anti-affinity/topology spread
-   Multiple availability zones
-   Proper readiness probes
-   Graceful shutdown
-   PDB
-   Reliable storage
-   Database HA
-   Backup/restore
-   Disaster recovery

Architecture:

``` text
              Load Balancer
                    |
          +---------+---------+
          |         |         |
        Pod A     Pod B     Pod C
          |         |         |
        Node 1    Node 2    Node 3
          |         |         |
        Zone A    Zone B    Zone C
```

## 12. Readiness vs Liveness

### Readiness

Answers:

> Can this pod receive traffic?

If false, the pod can remain running but should not receive normal
Service traffic.

### Liveness

Answers:

> Is this container/application alive enough to continue?

A failed liveness probe may cause a restart.

### Startup Probe

Useful for slow-starting applications.

``` text
Startup
   |
   v
startupProbe
   |
   v
readiness/liveness
```

## 13. Graceful Shutdown

Applications should handle termination correctly.

Kubernetes generally sends termination signals before forcefully killing
a container.

Applications should:

1.  Stop accepting new work.
2.  Finish or safely cancel active work.
3.  Close connections.
4.  Exit.

This is especially important for:

-   APIs
-   Message consumers
-   Workers
-   Long-running jobs

## 14. Capacity Planning

Capacity planning asks:

> Do we have enough resources for expected and unexpected demand?

Consider:

-   CPU
-   Memory
-   Network
-   Disk
-   Pod capacity
-   Node capacity
-   Database capacity
-   API rate limits

Monitor trends rather than only current usage.

## 15. Scaling

### Horizontal Pod Autoscaler

Scales replicas.

``` text
Traffic increases
      |
      v
CPU/request metric increases
      |
      v
HPA increases replicas
```

### Cluster Autoscaler

Adjusts node count where supported.

``` text
Pending pods
    |
    v
Insufficient node capacity
    |
    v
More nodes
```

Do not assume HPA alone solves every scaling problem.

## 16. Incident Runbook

A runbook should contain:

``` text
Symptom
Impact
Initial checks
Commands
Decision tree
Mitigation
Rollback
Escalation
Recovery verification
Post-incident actions
```

Example:

``` text
Symptom: HTTP 503

1. Check ingress
2. Check service
3. Check endpoints
4. Check pod readiness
5. Check application logs
6. Check recent deployment
7. Roll back if release-related
8. Confirm recovery
9. Record incident
```

## 17. Postmortem

A good postmortem should be blameless.

Include:

-   Summary
-   Impact
-   Timeline
-   Detection
-   Root/contributing causes
-   What went well
-   What went poorly
-   Corrective actions
-   Owners
-   Due dates

Avoid:

``` text
"Engineer X made a mistake."
```

Prefer:

``` text
"The deployment process allowed an unsafe configuration to reach production without validation."
```

Focus on improving the system.

## 18. Change Safety

Production reliability depends heavily on safe changes.

Useful techniques:

-   Rolling updates
-   Canary releases
-   Blue/green deployments
-   Automated tests
-   Security scanning
-   Image scanning
-   GitOps
-   Progressive delivery
-   Automatic rollback criteria

## 19. Error Budget and Releases

A mature organization connects reliability to delivery.

``` text
Healthy error budget
       |
       v
Normal release velocity

Low error budget
       |
       v
More reliability work

Budget exhausted
       |
       v
Restrict risky changes
```

The exact policy should be defined by the organization.

## 20. Disaster Recovery

Know the difference:

### Backup

A copy of data.

### Restore

Recovering data from a backup.

### Disaster Recovery

The broader process of restoring service after a major failure.

Important concepts:

-   RPO --- Recovery Point Objective
-   RTO --- Recovery Time Objective

### RPO

How much data loss is acceptable?

Example:

``` text
RPO = 15 minutes
```

### RTO

How quickly must service recover?

Example:

``` text
RTO = 1 hour
```

## 21. Practical Production Lab

Build:

``` text
Client
  |
Ingress
  |
Service
  |
Deployment
  |
Pods
  |
Database
```

Add:

``` text
Prometheus
Grafana
Alertmanager
```

Then simulate:

1.  Delete a pod.
2.  Break a readiness probe.
3.  Deploy a bad image.
4.  Increase application traffic.
5.  Exhaust memory.
6.  Remove a Service endpoint.
7.  Roll back the deployment.

For each scenario document:

``` text
Detection
SLI impact
SLO impact
Alert
Diagnosis
Mitigation
Recovery
Permanent fix
```

## 22. Interview Questions

### What is MTTR?

Mean Time To Recovery/Repair; it measures how quickly service is
restored after an incident.

### What is MTTD?

Mean Time To Detect; how quickly a failure is detected.

### Why are readiness probes important?

They prevent traffic from being sent to a pod that is not ready to serve
requests.

### What is a PDB?

A PodDisruptionBudget limits certain voluntary disruptions to maintain
application availability.

### What is RPO?

Maximum acceptable amount of data loss measured in time.

### What is RTO?

Target maximum time to restore service after a disruption.

### How do you troubleshoot HTTP 503?

Start at the traffic path:

``` text
Client
 -> Ingress
 -> Service
 -> EndpointSlice
 -> Pod
 -> Application
```

Inspect each layer and correlate with metrics, logs and traces.

## 23. Must Understand

You should be able to explain:

-   MTTD
-   MTTR
-   Incident lifecycle
-   Runbooks
-   Postmortems
-   Readiness/liveness/startup probes
-   PDB
-   High availability
-   Capacity planning
-   HPA
-   Cluster autoscaling
-   RPO/RTO
-   Disaster recovery
-   Safe deployments
-   Error-budget-driven reliability
