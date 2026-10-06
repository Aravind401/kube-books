# Kubernetes Phase 2.5 --- SLI, SLO, SLA and Error Budgets

## 1. The Reliability Hierarchy

``` text
Real user experience
        |
        v
       SLI
        |
        v
       SLO
        |
        v
  Error Budget
        |
        v
       SLA
```

These terms are related but are not interchangeable.

## 2. SLI --- Service Level Indicator

An SLI is a measurement of actual service behavior.

Examples:

-   Availability
-   Latency
-   Error rate
-   Throughput
-   Durability

### Availability SLI

``` text
successful requests
-------------------
total valid requests
```

Example:

``` text
9,990 successful
10,000 total

SLI = 99.9%
```

### Error Rate SLI

``` text
failed requests
---------------
total requests
```

Example:

``` text
50 failures
10,000 requests

Error rate = 0.5%
```

### Latency SLI

Example:

> Percentage of valid requests completed within 300 ms.

If 9,700 out of 10,000 requests complete within 300 ms:

``` text
Latency SLI = 97%
```

## 3. SLO --- Service Level Objective

An SLO is the reliability target.

Example:

``` text
Availability SLO = 99.9%
```

Meaning:

> The service should successfully serve 99.9% of qualifying requests
> during the defined measurement window.

Another example:

``` text
99% of requests must complete within 300 ms
```

## 4. SLA --- Service Level Agreement

An SLA is a business/customer commitment.

Example:

> The provider guarantees 99.9% monthly availability.

An SLA may include:

-   Contractual commitments
-   Service credits
-   Financial consequences
-   Customer obligations

### Important distinction

``` text
SLI = What actually happened?
SLO = What reliability do we target?
SLA = What do we contractually promise?
```

## 5. Error Budget

Error budget is the amount of unreliability allowed by the SLO.

If:

``` text
SLO = 99.9%
```

Then:

``` text
Error budget = 0.1%
```

For a 30-day month:

``` text
30 days × 24 hours × 60 minutes
= 43,200 minutes

0.1% × 43,200
= 43.2 minutes
```

So approximately **43 minutes 12 seconds** of unavailability is allowed
under a simple time-based 30-day model.

The exact calculation depends on the SLI definition and measurement
window.

## 6. Common Availability Targets

  SLO        Error Budget   Approx. 30-day time budget
  -------- -------------- ----------------------------
  99%                  1%                       7h 12m
  99.9%              0.1%                      43m 12s
  99.95%            0.05%                      21m 36s
  99.99%            0.01%                       4m 19s

Higher reliability is increasingly expensive.

## 7. SLO Is Not "100%"

100% reliability is usually unrealistic and expensive.

Instead:

``` text
Business requirement
        |
        v
Acceptable user impact
        |
        v
SLO
        |
        v
Engineering investment
```

The right SLO depends on:

-   Business criticality
-   Customer expectations
-   Cost
-   Architecture
-   Dependencies
-   Regulatory requirements

## 8. Error Budget Policy

An error budget should influence engineering decisions.

Example:

``` text
SLO = 99.9%

Budget remaining = 80%
    |
    +--> Continue feature delivery
    +--> Allow controlled releases

Budget remaining = 20%
    |
    +--> Increase reliability work
    +--> Review risky deployments

Budget exhausted
    |
    +--> Freeze non-essential risky changes
    +--> Focus on reliability
```

This should be an organizational policy rather than an automatic "no
deployments ever" rule.

## 9. Burn Rate

Burn rate tells us how quickly the error budget is being consumed.

``` text
Burn rate =
actual error rate
-----------------
allowed error rate
```

Example:

``` text
SLO = 99.9%
Allowed error rate = 0.1%

Actual error rate = 1%

Burn rate = 10x
```

At 10x burn rate, the budget is being consumed much faster than planned.

## 10. Fast Burn vs Slow Burn

### Fast burn

A severe incident causes rapid budget consumption.

Example:

``` text
5xx suddenly = 20%
```

This should trigger urgent alerting.

### Slow burn

A small degradation persists for a long time.

Example:

``` text
5xx = 0.3%
```

It may not look dramatic but can consume the budget over time.

A mature alerting strategy detects both.

## 11. Choosing Good SLIs

Bad SLI:

``` text
CPU < 80%
```

CPU is useful operational telemetry, but it may not directly represent
user experience.

Better user-facing SLI:

``` text
Successful checkout requests / valid checkout requests
```

Another:

``` text
Percentage of API requests completing under 500ms
```

The best SLIs usually represent something users care about.

## 12. Example Kubernetes API SLO

Suppose:

``` text
Service: Order API

Availability SLO:
99.9% successful requests over 30 days

Latency SLO:
99% of requests < 500ms

Error budget:
0.1% availability failures
```

Potential Prometheus-style signals:

``` text
total requests
5xx requests
request duration histogram
```

## 13. SLO Design Questions

Before creating an SLO ask:

1.  Who is the user?
2.  What action matters?
3.  What counts as success?
4.  What counts as failure?
5.  What measurement window?
6.  What target is appropriate?
7.  What alert threshold is useful?
8.  What happens when the error budget is exhausted?

## 14. Practical Exercise

Create SLOs for:

### Login API

-   Availability
-   Latency

### Payment API

-   Availability
-   Latency
-   Business success rate

### Static frontend

-   Availability
-   Page load performance

For each, define:

``` text
SLI
SLO
Error budget
Alert condition
Business impact
```

## 15. Interview Questions

### Q: SLI vs SLO?

SLI is the measured reliability indicator. SLO is the target for that
indicator.

### Q: SLO vs SLA?

SLO is generally an internal reliability objective. SLA is a
customer/business agreement with contractual implications.

### Q: What is an error budget?

The allowed amount of unreliability implied by the SLO.

### Q: Why not set every SLO to 99.99%?

Because reliability has a cost. The business may not need that level of
availability, and achieving it can reduce engineering velocity
significantly.

### Q: What is burn rate?

How quickly actual unreliability consumes the available error budget.

## 16. Must Understand

You should be able to explain:

``` text
SLI
 |
 +--> SLO
       |
       +--> Error Budget
               |
               +--> Burn Rate
                       |
                       +--> Reliability decisions

SLA = External/business commitment
```
