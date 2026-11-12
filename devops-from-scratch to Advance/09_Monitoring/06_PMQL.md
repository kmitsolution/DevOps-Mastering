# Lesson 6 — Metrics, Labels & Time Series

This is one of the **most important Prometheus lessons**.

If you understand **metrics + labels + time series**, PromQL will become much easier.

Today we'll use **Kubernetes examples**, because that's your main goal.

---

# 1. What is a Metric?

A metric is a measurement.

For example:

```text
CPU usage
Memory usage
Number of requests
Number of errors
Number of running Pods
```

Prometheus stores these measurements over time.

A simple metric could be:

```promql
up
```

You already saw this in Lesson 5.

It might produce:

```text
up 1
```

where:

```text
1 = target is available
0 = target is unavailable
```

But things become much more interesting when **labels** are added.

---

# 2. What Are Labels?

Labels are **key-value pairs attached to a metric**.

Example:

```promql
up{
  job="node-exporter",
  instance="10.0.0.10:9100"
}
```

Here:

```text
Metric:
up

Labels:
job      = node-exporter
instance = 10.0.0.10:9100

Value:
1
```

Think of labels as giving additional information about the metric.

---

# 3. Why Do We Need Labels?

Imagine Kubernetes has:

```text
10 Nodes
500 Pods
20 Namespaces
```

You don't want 500 different metric names like:

```text
pod1_cpu
pod2_cpu
pod3_cpu
...
```

Instead, Prometheus uses one metric name with labels.

For example:

```promql
container_cpu_usage_seconds_total{
    namespace="production",
    pod="api-123",
    container="api"
}
```

and:

```promql
container_cpu_usage_seconds_total{
    namespace="production",
    pod="api-456",
    container="api"
}
```

Same metric name.

Different labels.

---

# 4. Think of Labels Like Database Columns

This is a useful analogy.

Imagine a table:

| metric | namespace   | pod   | container | value |
| ------ | ----------- | ----- | --------- | ----: |
| CPU    | production  | api-1 | api       |    20 |
| CPU    | production  | api-2 | api       |    30 |
| CPU    | development | api-3 | api       |    10 |

Prometheus doesn't work exactly like a relational database, but this is a useful mental model.

You can think:

```text
Metric = measurement type

Labels = dimensions

Value = measurement
```

---

# 5. Example: Kubernetes Pod Metric

Let's look at:

```promql
kube_pod_status_phase
```

You might see something conceptually like:

```text
kube_pod_status_phase{
    namespace="default",
    pod="nginx-abc123",
    phase="Running"
} 1
```

Break it apart:

```text
Metric name
     │
     ▼
kube_pod_status_phase

Labels
     │
     ├── namespace = default
     ├── pod       = nginx-abc123
     └── phase     = Running

Value
     │
     ▼
1
```

---

# 6. What Is a Time Series?

This is the key concept.

A **time series is a sequence of values for one unique combination of metric name and labels over time.**

For example:

```text
kube_pod_status_phase{
    namespace="default",
    pod="nginx-abc123",
    phase="Running"
}
```

could have:

```text
Time       Value
----------------
10:00        1
10:01        1
10:02        1
10:03        1
```

That's one time series.

---

# 7. Change One Label

Now suppose we have another Pod:

```text
kube_pod_status_phase{
    namespace="default",
    pod="nginx-xyz789",
    phase="Running"
}
```

That's a **different time series**.

Why?

Because:

```text
pod="nginx-abc123"
```

is different from:

```text
pod="nginx-xyz789"
```

So:

```text
Same metric
     +
Different labels
     ↓
Different time series
```

---

# 8. A More Visual Example

Suppose we have:

```text
Namespace: production

Pods:
api-1
api-2
api-3
```

Prometheus may have:

```text
CPU{
  namespace="production",
  pod="api-1"
}

CPU{
  namespace="production",
  pod="api-2"
}

CPU{
  namespace="production",
  pod="api-3"
}
```

That's:

```text
             CPU metric
                 │
       ┌─────────┼─────────┐
       ▼         ▼         ▼
     api-1     api-2     api-3
       │         │         │
      20%       40%       70%
```

Three time series.

---

# 9. Labels Are Extremely Powerful

Suppose you want CPU for only:

```text
namespace = production
```

You can filter:

```promql
container_cpu_usage_seconds_total{
    namespace="production"
}
```

Or only:

```text
pod="api-1"
```

```promql
container_cpu_usage_seconds_total{
    pod="api-1"
}
```

This is the beginning of PromQL.

---

# 10. Label Matching

PromQL provides several ways to filter labels.

### Exact match

```promql
metric{namespace="production"}
```

Means:

> namespace exactly equals production.

---

### Not equal

```promql
metric{namespace!="production"}
```

Means:

> namespace is anything except production.

---

### Regular expression

```promql
metric{namespace=~"prod.*"}
```

Means:

> namespace starts with `prod`.

---

### Negative regular expression

```promql
metric{namespace!~"dev.*"}
```

Means:

> namespace doesn't match `dev.*`.

---

# 11. Let's Try It With `up`

Start with:

```promql
up
```

Now look at the labels.

You may see something like:

```text
up{
  instance="...",
  job="..."
} 1
```

Now filter:

```promql
up{job="node-exporter"}
```

This asks:

> Show only targets belonging to the `node-exporter` job.

---

# 12. Filter by Multiple Labels

You can combine labels.

For example:

```promql
up{
  job="node-exporter",
  instance="10.0.0.10:9100"
}
```

Meaning:

```text
job = node-exporter
AND
instance = 10.0.0.10:9100
```

Both conditions must match.

---

# 13. Metric Name Is Also a Selector

This:

```promql
up
```

is equivalent conceptually to selecting:

```text
__name__ = "up"
```

You can actually use:

```promql
{__name__="up"}
```

and you'll get the `up` metric.

This becomes useful when doing more advanced PromQL.

---

# 14. Labels in Kubernetes

Kubernetes monitoring generates many useful labels.

For example:

```text
namespace
pod
container
node
job
instance
service
endpoint
cluster
```

So a metric might look like:

```text
container_memory_usage_bytes{
    namespace="production",
    pod="frontend-7d8c9",
    container="frontend",
    node="worker-1"
}
```

Now you can ask questions like:

> How much memory is the frontend Pod using?

or:

> How much memory is being used by all Pods in production?

That's where aggregation comes in.

---

# 15. Multiple Time Series

Suppose:

```text
production namespace
```

contains:

```text
api-1
api-2
api-3
frontend-1
frontend-2
```

A query such as:

```promql
container_memory_usage_bytes{
    namespace="production"
}
```

might return:

```text
api-1       500 MB
api-2       600 MB
api-3       550 MB
frontend-1  300 MB
frontend-2  350 MB
```

Prometheus returned **five time series**.

---

# 16. Aggregation

Now you might want:

> Total memory used by production.

Use:

```promql
sum(
  container_memory_usage_bytes{
    namespace="production"
  }
)
```

Conceptually:

```text
500
+
600
+
550
+
300
+
350
────
2300 MB
```

Now you've converted multiple time series into one result.

---

# 17. `sum by`

This is one of the most important PromQL concepts.

Suppose:

```text
production
│
├── api-1
├── api-2
├── api-3
├── frontend-1
└── frontend-2
```

You want memory **per Pod**.

You might use:

```promql
sum by (pod) (
  container_memory_usage_bytes{
    namespace="production"
  }
)
```

Result:

```text
api-1        500 MB
api-2        600 MB
api-3        550 MB
frontend-1   300 MB
frontend-2   350 MB
```

---

# 18. `sum by (namespace)`

Now suppose we have:

```text
dev
production
testing
```

We can ask:

```promql
sum by (namespace) (
  container_memory_usage_bytes
)
```

Conceptually:

```text
dev         2 GB
production  8 GB
testing     1 GB
```

This is exactly the type of query you'll use to build Grafana dashboards.

---

# 19. Why Labels Matter So Much in Grafana

Imagine your Grafana dashboard has:

```text
Namespace:
[ production ▼ ]
```

When you select:

```text
production
```

Grafana can modify the PromQL query to filter:

```promql
{namespace="production"}
```

Then you select:

```text
development
```

and the query becomes:

```promql
{namespace="development"}
```

This is called a **Grafana variable**.

We'll build these later.

---

# 20. Metric + Labels + Value + Timestamp

Let's put everything together.

A Prometheus sample conceptually contains:

```text
Metric
   +
Labels
   +
Timestamp
   +
Value
```

Example:

```text
container_memory_usage_bytes{
    namespace="production",
    pod="api-1",
    container="api"
}
```

At:

```text
12:30:00
```

the value might be:

```text
524288000
```

So:

```text
Metric:
container_memory_usage_bytes

Labels:
namespace=production
pod=api-1
container=api

Timestamp:
12:30:00

Value:
524288000 bytes
```

That is a Prometheus sample.

---

# 21. Important: Don't Create Too Many Labels

There's a concept called **cardinality**.

Suppose you have:

```text
user_id
```

and your application has:

```text
10 million users
```

If you put:

```text
user_id="12345"
```

into a metric label, you could create an enormous number of time series.

For example:

```text
http_requests_total{
    user_id="1"
}

http_requests_total{
    user_id="2"
}

...

http_requests_total{
    user_id="10000000"
}
```

That's potentially disastrous for a metrics system.

This is called **high cardinality**.

---

# 22. Good vs Bad Labels

### Usually good

```text
namespace
pod
container
service
method
status_code
environment
```

### Potentially dangerous

```text
user_id
request_id
transaction_id
session_id
```

because these can have huge numbers of unique values.

We'll revisit cardinality when we discuss application metrics.

---

# 23. Metric Types

Now we're ready to introduce the four common Prometheus metric types.

```text
Metric Types
     │
     ├── Counter
     ├── Gauge
     ├── Histogram
     └── Summary
```

## Counter

Only goes up, except when the process restarts.

Example:

```text
http_requests_total
```

```text
100
120
150
180
```

Useful for:

```text
Requests
Errors
Events
```

---

# 24. Gauge

Can go up and down.

Example:

```text
memory_usage_bytes
```

```text
2 GB
3 GB
5 GB
4 GB
2 GB
```

Useful for:

```text
CPU
Memory
Temperature
Queue size
Active connections
```

---

# 25. Histogram

Used for distributions.

A classic example:

```text
HTTP request latency
```

Suppose requests take:

```text
10ms
20ms
30ms
50ms
100ms
500ms
```

A histogram lets you understand the distribution.

This becomes very important when we monitor:

```text
p50 latency
p90 latency
p95 latency
p99 latency
```

We'll study this carefully later.

---

# 26. Summary

A Summary also deals with distributions and quantiles.

For your Kubernetes/DevOps learning path, I recommend focusing first on:

```text
Counter
Gauge
Histogram
```

and learning Summary after you're comfortable with those.

---

# 27. Practical Exercise

Now open Prometheus:

```text
http://localhost:9090
```

### Exercise 1

Run:

```promql
up
```

Look at the labels.

Identify:

```text
job
instance
```

---

### Exercise 2

Run:

```promql
up{job="node-exporter"}
```

Observe how the results change.

---

### Exercise 3

Run:

```promql
up == 1
```

Compare it with:

```promql
up
```

---

### Exercise 4

Try:

```promql
count(up)
```

Then:

```promql
count(up == 1)
```

---

### Exercise 5 — Kubernetes

Try:

```promql
kube_pod_status_phase
```

You should get many time series.

Look at the labels.

Find:

```text
namespace
pod
phase
```

---

### Exercise 6

Filter one namespace:

```promql
kube_pod_status_phase{
  namespace="default"
}
```

If you don't have useful Pods in `default`, choose another namespace that exists in your cluster.

---

### Exercise 7

Filter Running Pods:

```promql
kube_pod_status_phase{
  phase="Running"
}
```

---

# 🎯 Lesson 6 Summary

Remember this formula:

```text
Metric Name
     +
Labels
     +
Timestamp
     +
Value
     =
Prometheus Sample
```

And:

```text
Metric + unique label set
             ↓
       Time Series
```

For Kubernetes:

```text
                         Prometheus
                              │
                     ┌────────┼────────┐
                     │        │        │
                   Node      Pod      App
                     │        │        │
                     ▼        ▼        ▼
                  Labels    Labels   Labels
                     │        │        │
                     └────────┼────────┘
                              ▼
                         Time Series
                              │
                            PromQL
                              │
                              ▼
                           Grafana
```

## ⭐ The key concept

Don't think:

> "Prometheus stores CPU."

Think:

> **"Prometheus stores many time series representing CPU measurements, differentiated by labels such as node, pod, container, namespace, and instance."**

That mental model will make the next lesson much easier.

---

### Next: Lesson 7 — PromQL Fundamentals

We'll finally start learning PromQL properly:

```text
up
↓
Selectors
↓
Comparison operators
↓
sum()
↓
avg()
↓
min()/max()
↓
count()
↓
sum by()
↓
rate()
↓
increase()
```

And we'll use **actual Kubernetes metrics from your cluster**, not generic examples.
