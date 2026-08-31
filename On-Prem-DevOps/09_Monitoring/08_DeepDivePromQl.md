# Lesson 8 — PromQL Deep Dive for Kubernetes

Today we'll go deeper into **PromQL**, especially the queries you'll actually use for Kubernetes monitoring.

By the end of this lesson, you should be able to write queries for:

```text
Kubernetes
│
├── Node CPU
├── Node Memory
├── Pod CPU
├── Pod Memory
├── Pod Restarts
├── Pod Status
├── Namespace usage
└── Target health
```

---

# 1. First: Instant Vector vs Range Vector

This is one of the most important PromQL concepts.

## Instant Vector

When you run:

```promql
up
```

Prometheus gives you the **latest sample** for each matching time series.

Conceptually:

```text
up
│
├── target-1 → 1
├── target-2 → 1
└── target-3 → 0
```

This is an **instant vector**.

---

## Range Vector

Now:

```promql
up[5m]
```

means:

> Give me the samples from the last 5 minutes.

Conceptually:

```text
target-1
│
├── 10:00 → 1
├── 10:01 → 1
├── 10:02 → 1
├── 10:03 → 1
└── 10:04 → 1
```

That's a **range vector**.

### Remember

```text
metric
   ↓
latest value

metric[5m]
   ↓
values during last 5 minutes
```

---

# 2. Why Do We Need Range Vectors?

Because functions like:

```promql
rate()
increase()
```

need historical samples.

For example:

```promql
rate(http_requests_total[5m])
```

The `[5m]` tells Prometheus:

> Look at the last five minutes to calculate the rate.

---

# 3. `rate()` — Very Important

Suppose:

```text
http_requests_total
```

has:

```text
10:00 → 1000
10:01 → 1100
10:02 → 1250
10:03 → 1400
```

The counter is increasing.

Run:

```promql
rate(http_requests_total[5m])
```

You get approximately:

```text
requests / second
```

So:

```text
rate()
   ↓
How quickly is the counter increasing?
```

---

# 4. `irate()`

You'll also encounter:

```promql
irate()
```

For example:

```promql
irate(http_requests_total[5m])
```

The important conceptual difference:

```text
rate()
 ↓
more stable rate over the selected range

irate()
 ↓
rate based primarily on the most recent samples
```

For dashboards, **`rate()` is usually the better default**.

For rapidly changing signals, `irate()` can sometimes be useful.

---

# 5. `increase()`

Suppose:

```text
http_requests_total
```

increases from:

```text
10,000
```

to:

```text
12,000
```

over five minutes.

Then:

```promql
increase(http_requests_total[5m])
```

returns approximately:

```text
2000
```

So:

```text
rate()
    → per-second rate

increase()
    → total increase during range
```

---

# 6. `sum()`

Suppose:

```text
CPU
│
├── Pod A → 0.2
├── Pod B → 0.4
└── Pod C → 0.3
```

Then:

```promql
sum(metric)
```

returns:

```text
0.9
```

It combines all matching series.

---

# 7. `sum by()`

Suppose:

```text
namespace
│
├── production
│     ├── api-1
│     └── api-2
│
└── development
      ├── api-3
      └── api-4
```

You can calculate usage by namespace:

```promql
sum by (namespace) (
  metric
)
```

Result:

```text
production   0.8
development  0.3
```

This is one of the most useful patterns in Grafana.

---

# 8. `sum without()`

The opposite-style operation is:

```promql
sum without(instance) (
  metric
)
```

Instead of saying:

> Group by these labels.

you're saying:

> Aggregate while removing this label from the grouping dimensions.

You don't need to use `without()` frequently at first, but it's useful when you understand more complex metric label sets.

---

# 9. `topk()`

Suppose you want:

> Which 5 Pods are using the most CPU?

Use:

```promql
topk(
  5,
  sum by (pod) (
    rate(container_cpu_usage_seconds_total[5m])
  )
)
```

Conceptually:

```text
Pod        CPU
───────────────
api-7      0.90
api-3      0.82
worker-2   0.75
api-5      0.70
worker-1   0.65
```

This is extremely useful for troubleshooting.

---

# 10. `bottomk()`

Similarly:

```promql
bottomk(
  5,
  sum by (pod) (
    rate(container_cpu_usage_seconds_total[5m])
  )
)
```

returns the five lowest values.

---

# 11. `max()`

Suppose you want the maximum memory consumption:

```promql
max(
  container_memory_usage_bytes
)
```

This gives you the highest value among the matching series.

---

# 12. `max by()`

More useful:

```promql
max by (namespace) (
  container_memory_usage_bytes
)
```

This tells you:

> Maximum memory usage within each namespace.

---

# 13. Kubernetes CPU Monitoring

Now let's start building real Kubernetes queries.

A commonly encountered metric is:

```promql
container_cpu_usage_seconds_total
```

This is a **counter**.

Therefore, don't normally display it directly as CPU usage.

Use:

```promql
rate(
  container_cpu_usage_seconds_total[5m]
)
```

Then aggregate by Pod:

```promql
sum by (pod) (
  rate(container_cpu_usage_seconds_total[5m])
)
```

---

# 14. CPU by Namespace

Now:

```promql
sum by (namespace) (
  rate(container_cpu_usage_seconds_total[5m])
)
```

You might see:

```text
namespace       CPU
--------------------
default         0.25
monitoring      1.50
production      2.20
```

Interpretation:

> The production namespace is currently consuming the most CPU among these results.

---

# 15. CPU by Container

You can also group by:

```promql
sum by (container) (
  rate(container_cpu_usage_seconds_total[5m])
)
```

This gives:

```text
container      CPU
------------------
api            1.2
frontend       0.5
redis          0.3
```

---

# 16. CPU by Node

Depending on the metrics available in your installation, you can aggregate by node:

```promql
sum by (node) (
  rate(container_cpu_usage_seconds_total[5m])
)
```

Conceptually:

```text
Node       CPU
───────────────
worker-1   2.1
worker-2   1.7
worker-3   3.2
```

This becomes useful for identifying overloaded nodes.

---

# 17. Kubernetes Memory

Memory is different.

A commonly encountered metric is:

```promql
container_memory_working_set_bytes
```

This is a **gauge**.

Therefore:

```text
Don't use rate()
```

for ordinary current memory usage.

Try:

```promql
container_memory_working_set_bytes
```

Then:

```promql
sum by (pod) (
  container_memory_working_set_bytes
)
```

---

# 18. Memory by Namespace

Try:

```promql
sum by (namespace) (
  container_memory_working_set_bytes
)
```

You might see:

```text
namespace       Memory
-----------------------
default          1.2 GB
monitoring       4.8 GB
production       7.2 GB
```

This is an excellent Grafana panel.

---

# 19. Memory by Node

Similarly:

```promql
sum by (node) (
  container_memory_working_set_bytes
)
```

This helps you identify:

> Which Kubernetes node is consuming the most container memory?

---

# 20. Pod Restart Monitoring

Now an important operational metric.

Try:

```promql
kube_pod_container_status_restarts_total
```

This is generally a counter.

You can find Pods with recent restart activity using:

```promql
increase(
  kube_pod_container_status_restarts_total[1h]
)
```

Conceptually:

```text
Pod          Restarts/hour
──────────────────────────
api-1             0
api-2             3
worker-1          15
```

Now you have a troubleshooting signal.

---

# 21. Find Pods With Restarts

You can filter:

```promql
increase(
  kube_pod_container_status_restarts_total[1h]
) > 0
```

Meaning:

> Show containers that have restarted during the last hour.

This can eventually become an alert.

---

# 22. Pod Status

We've already used:

```promql
kube_pod_status_phase
```

Try:

```promql
kube_pod_status_phase{
  phase="Running"
}
```

Then:

```promql
count(
  kube_pod_status_phase{
    phase="Running"
  }
)
```

And:

```promql
count by (namespace) (
  kube_pod_status_phase{
    phase="Running"
  }
)
```

---

# 23. Detect Non-Running Pods

You can inspect:

```promql
kube_pod_status_phase{
  phase!="Running"
}
```

But be careful.

Kubernetes phase metrics represent several states, and filtering alone can produce multiple zero-valued series.

For operational queries, we'll refine these expressions later.

The important lesson today is:

> **Understand the metric's labels and semantics before writing an alert.**

---

# 24. `absent()`

Another useful function:

```promql
absent(up)
```

It answers:

> Does this metric completely not exist?

This is different from:

```promql
up == 0
```

because:

```text
up == 0
```

means:

> The target exists but is currently down.

while:

```text
absent(up)
```

means:

> No matching `up` time series exists at all.

This distinction becomes useful in alerting.

---

# 25. Operators

PromQL supports arithmetic.

For example:

```promql
metric1 + metric2
```

```promql
metric1 - metric2
```

```promql
metric1 * 100
```

```promql
metric1 / metric2
```

This becomes extremely useful for calculating percentages.

---

# 26. CPU Percentage — Concept

Suppose:

```text
CPU used
────────
CPU capacity
```

You can calculate:

```text
CPU utilization =
    used / capacity × 100
```

PromQL often involves this pattern:

```promql
(
  used
  /
  capacity
) * 100
```

We'll build production-quality CPU percentage queries later because Kubernetes CPU metrics can be tricky depending on the monitoring stack.

---

# 27. Memory Percentage — Concept

Similarly:

```text
Memory utilization =
    used / total × 100
```

Conceptually:

```promql
(
  used_memory
  /
  total_memory
) * 100
```

This is the type of calculation you'll eventually put into a Grafana gauge.

---

# 28. A Very Important PromQL Skill

When a query gives unexpected results:

**Don't immediately change the query.**

First inspect the raw metric.

For example:

```promql
container_memory_working_set_bytes
```

Look at:

```text
namespace
pod
container
node
instance
job
```

Then decide:

> Which labels should I keep?

Then aggregate.

For example:

```text
Raw metric
    ↓
Filter
    ↓
rate() if required
    ↓
sum by()
    ↓
percentage/calculation
    ↓
Grafana
```

---

# 29. PromQL Workflow

This is the workflow I want you to develop.

Suppose the question is:

> Which namespace uses the most CPU?

### Step 1 — Find the metric

```promql
container_cpu_usage_seconds_total
```

### Step 2 — Recognize the type

It's a counter.

### Step 3 — Calculate rate

```promql
rate(container_cpu_usage_seconds_total[5m])
```

### Step 4 — Group

```promql
sum by (namespace) (
  rate(container_cpu_usage_seconds_total[5m])
)
```

### Step 5 — Find highest

```promql
topk(
  5,
  sum by (namespace) (
    rate(container_cpu_usage_seconds_total[5m])
  )
)
```

This is how you should approach PromQL problems.

---

# 🧪 Lesson 8 Practical Lab

Open:

```text
http://localhost:9090
```

Run these.

### 1. Targets

```promql
up
```

### 2. Count targets

```promql
count(up)
```

### 3. Running Pods

```promql
kube_pod_status_phase{
  phase="Running"
}
```

### 4. Running Pods by namespace

```promql
count by (namespace) (
  kube_pod_status_phase{
    phase="Running"
  }
)
```

### 5. Memory by Pod

```promql
sum by (pod) (
  container_memory_working_set_bytes
)
```

### 6. Memory by namespace

```promql
sum by (namespace) (
  container_memory_working_set_bytes
)
```

### 7. CPU by Pod

```promql
sum by (pod) (
  rate(container_cpu_usage_seconds_total[5m])
)
```

### 8. CPU by namespace

```promql
sum by (namespace) (
  rate(container_cpu_usage_seconds_total[5m])
)
```

### 9. Top 5 CPU Pods

```promql
topk(
  5,
  sum by (pod) (
    rate(container_cpu_usage_seconds_total[5m])
  )
)
```

### 10. Pod restarts

```promql
increase(
  kube_pod_container_status_restarts_total[1h]
)
```

---

# 🧠 Your Kubernetes PromQL Cheat Sheet

Keep this mental table:

| Requirement             | PromQL concept                                           |
| ----------------------- | -------------------------------------------------------- |
| Target health           | `up`                                                     |
| Count                   | `count()`                                                |
| Total                   | `sum()`                                                  |
| Average                 | `avg()`                                                  |
| Highest                 | `max()`                                                  |
| Lowest                  | `min()`                                                  |
| Group                   | `sum by()`                                               |
| Counter rate            | `rate()`                                                 |
| Recent counter increase | `increase()`                                             |
| Fast recent rate        | `irate()`                                                |
| Top N                   | `topk()`                                                 |
| Bottom N                | `bottomk()`                                              |
| Missing metric          | `absent()`                                               |
| CPU                     | `rate(container_cpu_usage_seconds_total[5m])`            |
| Memory                  | `container_memory_working_set_bytes`                     |
| Pod restarts            | `increase(kube_pod_container_status_restarts_total[1h])` |
| Pod state               | `kube_pod_status_phase`                                  |

---

# 🎯 Challenge

Try writing these yourself before looking at the answers.

### Challenge 1

**Show the top 3 Pods by memory usage.**

Hint:

```text
topk
sum by
container_memory_working_set_bytes
```

### Challenge 2

**Show the top 5 namespaces by CPU consumption.**

Hint:

```text
topk
sum by (namespace)
rate
```

### Challenge 3

**Show Pods that have restarted during the last hour.**

Hint:

```text
increase(...) > 0
```

### Challenge 4

**Show all targets that are down.**

### Challenge 5

**Show the number of Running Pods per namespace.**

---

# 🔥 One Important Warning

You may notice that some of the Kubernetes CPU/memory queries return things like:

```text
pod=""
container=""
namespace=""
```

or include system/infrastructure containers.

**Don't "fix" this randomly yet.**

This is actually an important part of Kubernetes PromQL.

In the next lesson we'll learn:

# **Lesson 9 — Advanced PromQL + Kubernetes Filtering**

We'll learn how to properly filter:

```text
namespace
pod
container
node
job
instance
```

and remove unwanted series.

Then we'll build clean queries such as:

```text
Top 5 Pods by CPU
Top 5 Pods by Memory
CPU by Namespace
Memory by Namespace
Node CPU %
Node Memory %
Pod Restart Rate
Pods Not Ready
```

After that, we're ready to move into **Grafana dashboards**, where these queries become actual visualizations.
