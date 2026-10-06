# Kubernetes Phase 2.5 --- Reliability, SLI/SLO/SLA & Observability

This phase extends the Kubernetes roadmap from networking/storage into
production reliability.

## Files

1.  `01-observability-fundamentals.md`
2.  `02-sli-slo-sla-error-budget.md`
3.  `03-prometheus-grafana-alerting.md`
4.  `04-reliability-incident-response.md`

## Recommended order

Observability → SLI/SLO/SLA → Prometheus/Grafana/Alerting →
Reliability/Incident Response

## End goal

You should be able to connect:

``` text
User Experience
      ↓
SLI
      ↓
SLO
      ↓
Error Budget
      ↓
Prometheus Metrics
      ↓
Grafana / Alertmanager
      ↓
Incident Response
      ↓
Reliability Improvements
```
