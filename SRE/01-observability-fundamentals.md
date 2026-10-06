# Kubernetes Phase 2.5 --- Observability Fundamentals

## 1. What is Observability?

Observability is the ability to understand the internal state of a
system by examining the data it produces.

The three primary telemetry signals are:

-   **Metrics** --- numerical measurements over time
-   **Logs** --- discrete records of events
-   **Traces** --- request-level journey across services

A production Kubernetes platform should ideally provide all three.

``` text
                    Production System
                           |
          +----------------+----------------+
          |                |                |
       Metrics           Logs             Traces
          |                |                |
     Prometheus       Loki/ELK/etc.      Jaeger/
          |                              Tempo/etc.
          +----------------+----------------+
                           |
                       Grafana
                           |
                    Human understanding
```

## 2. Monitoring vs Observability

### Monitoring

Monitoring asks:

> "Is the system behaving according to known expectations?"

Examples:

-   CPU \> 80%
-   Pod restart count increased
-   Disk usage \> 85%
-   HTTP 5xx \> 5%

### Observability

Observability asks:

> "Why is the system behaving this way?"

Example:

``` text
Users report slow checkout
        |
        v
Latency metric increases
        |
        v
Trace shows checkout -> payment-service
        |
        v
Payment service waits for database
        |
        v
DB connection pool exhausted
```

Monitoring detects the symptom. Observability helps investigate the
cause.

## 3. Metrics

Metrics are numerical time-series data.

Common Kubernetes metrics:

-   CPU usage
-   Memory usage
-   Pod restarts
-   Container network traffic
-   API request count
-   HTTP request latency
-   HTTP error count
-   Node filesystem usage
-   Kubernetes API server latency

### Four important metric types

#### Counter

Only increases, except when the process restarts.

Example:

``` text
http_requests_total
```

Use `rate()` or `increase()` in PromQL.

#### Gauge

Can increase or decrease.

Examples:

``` text
node_memory_available_bytes
kube_pod_status_ready
```

#### Histogram

Measures distributions such as latency.

Example:

``` text
http_request_duration_seconds_bucket
```

Useful for calculating percentiles and SLOs.

#### Summary

Client-side calculated quantiles.

Use carefully because summaries are generally harder to aggregate across
instances.

## 4. Logs

Logs describe events.

Example:

``` text
2026-10-06T20:30:10Z ERROR payment-service
request_id=abc123
database connection timeout
```

Good production logs should contain:

-   Timestamp
-   Severity
-   Service name
-   Environment
-   Request/correlation ID
-   Relevant resource ID
-   Error information

Avoid logging:

-   Passwords
-   Tokens
-   API keys
-   Sensitive personal information

## 5. Distributed Tracing

A trace represents one request across multiple services.

``` text
Trace: 8f92abc

frontend
   |
   +--> API Gateway
          |
          +--> order-service
                   |
                   +--> payment-service
                   |
                   +--> inventory-service
```

Each operation is a **span**.

Tracing is especially useful for:

-   Microservices
-   High latency
-   Dependency failures
-   Database calls
-   External API calls

Common tools include OpenTelemetry, Jaeger and Grafana Tempo.

## 6. Kubernetes Observability Stack

A typical stack:

``` text
Kubernetes
   |
   +--> Metrics --> Prometheus
   |                  |
   |                  +--> Grafana
   |
   +--> Logs ----> Loki / ELK
   |
   +--> Traces --> Tempo / Jaeger
   |
   +--> Alerts --> Alertmanager
```

## 7. RED Method

RED is especially useful for request-driven services.

### Rate

How many requests are arriving?

``` text
requests / second
```

### Errors

How many requests are failing?

``` text
5xx / total requests
```

### Duration

How long do requests take?

``` text
p50
p95
p99
```

Example dashboard:

``` text
Rate       850 req/s
Errors     0.7%
p95        240 ms
p99        680 ms
```

## 8. USE Method

USE is commonly applied to infrastructure resources.

### Utilization

How busy is the resource?

### Saturation

How much work is queued or unable to proceed?

### Errors

How many errors occurred?

For a node:

``` text
CPU utilization
CPU saturation/load
CPU errors

Memory utilization
Memory pressure
Memory errors

Disk utilization
Disk I/O queue
Disk errors
```

## 9. Cardinality

Cardinality is the number of unique combinations of metric labels.

Bad:

``` text
http_requests_total{
  user_id="123456",
  request_id="abc123"
}
```

If millions of users exist, metric cardinality can explode.

Better:

``` text
http_requests_total{
  service="checkout",
  method="POST",
  route="/orders",
  status="500"
}
```

Avoid high-cardinality labels such as:

-   user ID
-   request ID
-   session ID
-   raw URL
-   email address

## 10. Dashboards

A good dashboard should answer:

1.  Is the service healthy?
2.  Are users affected?
3.  What changed?
4.  Where is the bottleneck?
5.  What should I investigate next?

Recommended dashboard layers:

``` text
Executive
   |
Service health
   |
Application
   |
Kubernetes
   |
Node/infrastructure
```

## 11. Practical Lab

Deploy an application:

``` bash
kubectl create deployment web --image=nginx
kubectl expose deployment web --port=80
```

Inspect:

``` bash
kubectl get pods
kubectl get svc
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl top pod
kubectl top node
```

Questions:

-   Is the pod healthy?
-   How much CPU does it use?
-   How much memory does it use?
-   What happens when the pod restarts?
-   How would you detect this automatically?

## 12. Must Understand

You should be able to explain:

-   Monitoring vs observability
-   Metrics vs logs vs traces
-   Counter vs gauge vs histogram
-   RED method
-   USE method
-   Metric cardinality
-   Kubernetes observability architecture
-   Why request IDs matter
-   Why distributed tracing is useful
