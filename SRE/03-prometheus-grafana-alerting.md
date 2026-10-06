# Kubernetes Phase 2.5 --- Prometheus, Grafana and Alerting

## 1. Observability Architecture

A common Kubernetes monitoring architecture:

``` text
Kubernetes workloads
        |
        v
  Metrics endpoints
        |
        v
    Prometheus
        |
   +----+----+
   |         |
   v         v
Grafana   Alertmanager
   |         |
   v         v
Dashboards Notifications
```

## 2. Prometheus

Prometheus is a monitoring and time-series database system.

Key concepts:

-   Targets
-   Scraping
-   Time series
-   Labels
-   PromQL
-   Rules
-   Alerting
-   Exporters

## 3. Pull Model

Prometheus generally pulls metrics from targets.

``` text
Prometheus
    |
    | GET /metrics
    v
Application
```

This differs from systems where applications continuously push metrics.

## 4. Example Metric

``` text
http_requests_total{
  service="orders",
  method="GET",
  route="/orders",
  status="200"
}
```

A time series is identified by its metric name plus label set.

## 5. PromQL Basics

### Rate

``` promql
rate(http_requests_total[5m])
```

Requests per second over the recent 5-minute range.

### Total increase

``` promql
increase(http_requests_total[1h])
```

### Error rate

``` promql
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m]))
```

### CPU usage

A common node/container pattern:

``` promql
rate(container_cpu_usage_seconds_total[5m])
```

The exact query depends on the metric source and environment.

## 6. Histograms and Latency

Histogram metrics often expose:

``` text
request_duration_seconds_bucket
request_duration_seconds_sum
request_duration_seconds_count
```

A p95 approximation can be calculated using:

``` promql
histogram_quantile(
  0.95,
  sum by (le) (
    rate(http_request_duration_seconds_bucket[5m])
  )
)
```

## 7. Recording Rules

Recording rules precompute expensive or frequently used queries.

Example concept:

``` yaml
groups:
- name: service-recording
  rules:
  - record: service:http_requests_per_second
    expr: sum(rate(http_requests_total[5m]))
```

Benefits:

-   Faster dashboards
-   Reusable metrics
-   Reduced query cost

## 8. Alerting Rules

Example:

``` yaml
groups:
- name: application-alerts
  rules:
  - alert: HighErrorRate
    expr: |
      (
        sum(rate(http_requests_total{status=~"5.."}[5m]))
        /
        sum(rate(http_requests_total[5m]))
      ) > 0.05
    for: 10m
    labels:
      severity: critical
    annotations:
      summary: High HTTP error rate
```

The exact metric names depend on your instrumentation.

## 9. Alertmanager

Prometheus evaluates alert rules.

Alertmanager handles notification workflows.

``` text
Prometheus
    |
    v
Alert
    |
    v
Alertmanager
    |
 +--+-----+------+
 |        |      |
Slack   Email   Pager
```

Typical capabilities:

-   Grouping
-   Deduplication
-   Routing
-   Silencing
-   Inhibition

## 10. Good Alerting

A good alert should be:

-   Actionable
-   User-impact oriented
-   Clearly described
-   Routed to the correct team
-   Resistant to noise

Bad:

``` text
CPU > 80%
```

Better:

``` text
Order API has sustained high latency
and is violating its latency SLO.
```

Infrastructure alerts are still useful, but not every infrastructure
symptom should page an engineer.

## 11. SLO-Based Alerting

Instead of only:

``` text
CPU > 80%
```

consider:

``` text
SLO error budget is burning too quickly
```

This connects alerting to reliability.

## 12. Grafana

Grafana visualizes metrics and other telemetry.

Useful dashboard sections:

``` text
Service Overview
----------------
Requests/sec
Error rate
p95 latency
p99 latency

Kubernetes
----------
Pod count
Restarts
CPU
Memory

Infrastructure
--------------
Node CPU
Node memory
Disk
Network
```

## 13. Dashboard Design

A dashboard should move from:

``` text
Business impact
      |
      v
Service health
      |
      v
Application details
      |
      v
Kubernetes
      |
      v
Infrastructure
```

Do not create a dashboard containing hundreds of unrelated graphs.

## 14. Exporters

Exporters expose metrics for systems that do not natively expose
Prometheus metrics.

Examples:

-   Node Exporter
-   kube-state-metrics
-   Blackbox Exporter

### kube-state-metrics

Provides metrics based on Kubernetes object state.

Examples:

-   Deployment replicas
-   Pod status
-   StatefulSet state
-   Job status

It does not measure everything happening inside the application itself.

## 15. Kubernetes Monitoring Stack

A typical stack can include:

``` text
Prometheus
kube-state-metrics
node-exporter
Grafana
Alertmanager
```

## 16. Practical Lab

Install a monitoring stack using your preferred package manager/chart.

Then investigate:

``` bash
kubectl get pods -A
kubectl get svc -A
kubectl get nodes
kubectl top nodes
kubectl top pods -A
```

Create a simple application and expose metrics.

Verify:

``` text
Application
    |
    v
/metrics
    |
    v
Prometheus
    |
    v
PromQL
    |
    v
Grafana
```

## 17. Troubleshooting

### Prometheus cannot scrape target

Check:

``` bash
kubectl get pods
kubectl get svc
kubectl describe pod <pod>
kubectl get endpoints
kubectl get endpointslices
```

Then verify:

-   Port
-   Service
-   Labels
-   NetworkPolicy
-   Target configuration
-   Application metrics endpoint

### Grafana has no data

Check:

1.  Prometheus is healthy.
2.  Grafana datasource is configured.
3.  PromQL returns data in Prometheus.
4.  Dashboard time range is correct.

### Too many alerts

Review:

-   Alert thresholds
-   Alert duration
-   Grouping
-   SLO relevance
-   Duplicate alerts
-   Alert ownership

## 18. Must Understand

You should be able to explain:

-   Prometheus architecture
-   Pull/scrape model
-   PromQL
-   Labels
-   Cardinality
-   Histograms
-   Recording rules
-   Alert rules
-   Alertmanager
-   Grafana
-   kube-state-metrics
-   Exporters
-   SLO-based alerting
