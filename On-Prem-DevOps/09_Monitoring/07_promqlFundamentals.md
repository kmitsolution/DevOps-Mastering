# Lesson 7 — PromQL Fundamentals

Today we start one of the **most important parts of Prometheus**: **PromQL (Prometheus Query Language)**.

Our goal is not just to memorize queries. We want you to learn how to answer questions such as:

> Are my Kubernetes targets healthy?

> How many Pods are running?

> Which Pods are consuming the most CPU?

> How much memory is each namespace using?

> How many requests are coming into my application?

Eventually, these same queries will power your **Grafana dashboards and alerts**.

---

# 1. What is PromQL?

PromQL is the query language used to retrieve and calculate data stored in Prometheus.

Think:

```text
Prometheus
    │
    │ stores metrics
    ▼
   TSDB
    │
    │ PromQL
    ▼
Result
```

Grafana also uses PromQL when Prometheus is configured as its data source.

---

# 2. Start With the Simplest Query

Open:

```text
http://localhost:9090
```

Run:

```promql
up
```

You should get multiple results.

Conceptually:

```text
up{job="prometheus"}             1
up{job="node-exporter"}          1
up{job="kube-state-metrics"}     1
```

Remember:

```text
1 = target is up
0 = target is down
```

---

# 3. Metric Selector

When you write:

```promql
up
```

you're asking:

> Give me all time series for the `up` metric.

You can think of it as:

```text
Metric name
    ↓
   up
```

---

# 4. Selecting by Labels

From Lesson 6:

```promql
up{
  job="node-exporter"
}
```

means:

> Give me `up` metrics where the `job` label equals `node-exporter`.

Try:

```promql
up{job="node-exporter"}
```

You should see the Node Exporter targets.

---

# 5. Multiple Label Filters

You can specify multiple labels:

```promql
up{
  job="node-exporter",
  instance="10.0.0.10:9100"
}
```

The meaning is:

```text
job = node-exporter
AND
instance = 10.0.0.10:9100
```

Both must match.

---

# 6. Comparison Operators

PromQL supports comparison operators:

```text
=
!=
>
<
>=
<=
```

For example:

```promql
up == 1
```

means:

> Show targets whose value is 1.

And:

```promql
up == 0
```

means:

> Show targets whose value is 0.

This becomes extremely useful for alerts.

---

# 7. `count()`

Suppose:

```promql
up
```

returns:

```text
10 targets
```

You can count them:

```promql
count(up)
```

Result:

```text
10
```

So:

```text
up
 ↓
multiple time series
 ↓
count()
 ↓
one number
```

---

# 8. `sum()`

Suppose:

```promql
up
```

returns:

```text
1
1
1
0
1
```

Then:

```promql
sum(up)
```

returns:

```text
4
```

Because:

```text
1 + 1 + 1 + 0 + 1 = 4
```

This can be useful for counting healthy targets.

---

# 9. Count Healthy Targets

You could use:

```promql
sum(up == 1)
```

Conceptually:

```text
UP targets
1
1
1
0
1

sum = 4
```

So you have:

```text
4 healthy targets
```

---

# 10. `avg()`

You can calculate an average:

```promql
avg(up)
```

Suppose:

```text
1
1
1
0
```

Then:

```text
average = 0.75
```

Although `avg(up)` isn't usually the most useful Kubernetes health query, it demonstrates how PromQL aggregation works.

---

# 11. `min()` and `max()`

Try:

```promql
min(up)
```

and:

```promql
max(up)
```

If all targets are healthy:

```text
min = 1
max = 1
```

If at least one target is down:

```text
min = 0
max = 1
```

Interesting, right?

You can use:

```promql
min(up)
```

as a simple way of asking:

> Is at least one target down?

---

# 12. `sum by()`

This is one of the **most important PromQL concepts**.

Suppose we have:

```text
Namespace     Pod
---------------------
default       api-1
default       api-2
production    api-3
production    api-4
```

You can aggregate by namespace:

```promql
sum by (namespace) (
  some_metric
)
```

The result might be:

```text
default       2
production    2
```

Instead of getting one giant number, Prometheus groups the results by the specified label.

---

# 13. `count by()`

Similarly:

```promql
count by (namespace) (
  kube_pod_info
)
```

can answer:

> How many Pods are associated with each namespace?

Conceptually:

```text
default       3
monitoring    15
production    8
```

This is already becoming useful for a Grafana dashboard.

---

# 14. Kubernetes Example — Pod Status

Let's use a real Kubernetes metric:

```promql
kube_pod_status_phase
```

Run:

```promql
kube_pod_status_phase
```

You'll see multiple series.

Conceptually:

```text
kube_pod_status_phase{
    namespace="default",
    pod="nginx-123",
    phase="Running"
} 1
```

---

# 15. Find Running Pods

Try:

```promql
kube_pod_status_phase{
  phase="Running"
}
```

This filters the metric based on the `phase` label.

You can then count them:

```promql
count(
  kube_pod_status_phase{
    phase="Running"
  }
)
```

This gives you the number of matching Running-Pod time series.

---

# 16. Running Pods by Namespace

Now:

```promql
count by (namespace) (
  kube_pod_status_phase{
    phase="Running"
  }
)
```

You might get:

```text
namespace       value
----------------------
default            3
monitoring        15
kube-system       10
```

This is a perfect candidate for a Grafana visualization later.

---

# 17. Important Kubernetes Detail

Be careful with metrics such as:

```promql
kube_pod_status_phase
```

They represent **state series**, not simply a database table saying "there is one Pod."

For example, a Pod can have multiple phase series:

```text
phase="Pending"    0
phase="Running"    1
phase="Succeeded"  0
phase="Failed"     0
phase="Unknown"    0
```

So filtering:

```promql
phase="Running"
```

is important when you want Running Pods.

---

# 18. `rate()`

Now we reach a very important PromQL function.

Suppose we have:

```promql
http_requests_total
```

This is normally a **Counter**.

It continuously increases:

```text
100
150
210
280
350
```

But you probably don't want:

> Total requests since the application started.

You want:

> Requests per second.

That's where `rate()` comes in.

```promql
rate(http_requests_total[5m])
```

Meaning approximately:

> Calculate the per-second rate of increase during the last 5 minutes.

---

# 19. Why `[5m]`?

This:

```promql
http_requests_total
```

is an instant vector.

This:

```promql
http_requests_total[5m]
```

means:

> Give me the samples from the last 5 minutes.

Then:

```promql
rate(http_requests_total[5m])
```

calculates the rate over that period.

Think:

```text
Metric
  ↓
Last 5 minutes of samples
  ↓
rate()
  ↓
Requests / second
```

---

# 20. `increase()`

Another useful function:

```promql
increase(http_requests_total[5m])
```

asks:

> How much did this counter increase during the last 5 minutes?

For example:

```text
Requests at start = 10,000
Requests at end   = 10,600

increase = 600
```

Compare:

```promql
rate(http_requests_total[5m])
```

→ approximately requests/sec

while:

```promql
increase(http_requests_total[5m])
```

→ requests added during 5 minutes.

---

# 21. `rate()` vs `increase()`

Remember this:

```text
rate()
   ↓
How fast is it increasing?

increase()
   ↓
How much did it increase?
```

Example:

```text
http_requests_total
```

Use:

```promql
rate(http_requests_total[5m])
```

for:

> Requests/sec

Use:

```promql
increase(http_requests_total[5m])
```

for:

> Requests during the last 5 minutes

---

# 22. `sum by()` + `rate()`

Now combine what we've learned.

Suppose you have:

```text
http_requests_total
```

with:

```text
namespace
pod
```

You can calculate request rate by Pod:

```promql
sum by (pod) (
  rate(http_requests_total[5m])
)
```

Or by namespace:

```promql
sum by (namespace) (
  rate(http_requests_total[5m])
)
```

This is exactly the type of query you'll eventually put into Grafana.

---

# 23. A Useful PromQL Pattern

You should start recognizing this structure:

```promql
aggregation(
    function(
        metric{labels}[time]
    )
)
```

For example:

```promql
sum by (namespace) (
  rate(http_requests_total[5m])
)
```

Break it down:

```text
http_requests_total
        ↓
     [5m]
        ↓
      rate()
        ↓
   sum by namespace
```

This pattern appears everywhere in Prometheus monitoring.

---

# 24. Let's Apply This to Kubernetes CPU

One commonly encountered container CPU metric is:

```promql
container_cpu_usage_seconds_total
```

It is a counter.

So instead of directly looking at it, we calculate its rate:

```promql
rate(
  container_cpu_usage_seconds_total[5m]
)
```

Then we can aggregate.

For example:

```promql
sum by (pod) (
  rate(container_cpu_usage_seconds_total[5m])
)
```

This can give you CPU consumption rate by Pod.

**Don't worry if your exact result contains infrastructure/system containers or empty labels.** We'll refine Kubernetes queries in a later lesson.

---

# 25. Memory Is Different

Memory usage is generally a **Gauge**, not a Counter.

So you don't use:

```promql
rate()
```

for normal current memory usage.

You can query it directly:

```promql
container_memory_usage_bytes
```

or aggregate:

```promql
sum by (pod) (
  container_memory_usage_bytes
)
```

This illustrates an important rule:

```text
Counter → often use rate()

Gauge → usually query directly
```

---

# 26. PromQL Cheat Sheet — Lesson 7

| Goal               | Query                        |
| ------------------ | ---------------------------- |
| All targets        | `up`                         |
| Healthy targets    | `up == 1`                    |
| Down targets       | `up == 0`                    |
| Count targets      | `count(up)`                  |
| Sum values         | `sum(metric)`                |
| Average            | `avg(metric)`                |
| Minimum            | `min(metric)`                |
| Maximum            | `max(metric)`                |
| Group by namespace | `sum by (namespace)(metric)` |
| Group by pod       | `sum by (pod)(metric)`       |
| Counter rate       | `rate(metric[5m])`           |
| Counter increase   | `increase(metric[5m])`       |

---

# 🧪 Lesson 7 Practical Lab

Open:

```text
http://localhost:9090
```

Run these **in order**.

### Exercise 1

```promql
up
```

---

### Exercise 2

```promql
up == 1
```

---

### Exercise 3

```promql
up == 0
```

---

### Exercise 4

```promql
count(up)
```

---

### Exercise 5

```promql
sum(up)
```

Compare Exercises 4 and 5.

---

### Exercise 6

```promql
kube_pod_status_phase{
  phase="Running"
}
```

---

### Exercise 7

```promql
count(
  kube_pod_status_phase{
    phase="Running"
  }
)
```

---

### Exercise 8

```promql
count by (namespace) (
  kube_pod_status_phase{
    phase="Running"
  }
)
```

---

### Exercise 9

Try:

```promql
sum by (namespace) (
  container_memory_usage_bytes
)
```

Look at the results.

---

### Exercise 10

Try:

```promql
sum by (pod) (
  rate(container_cpu_usage_seconds_total[5m])
)
```

Look at which Pods are consuming CPU.

---

# 🎯 Challenge

Now try to answer these **without looking at the previous queries**.

### Challenge 1

> Show me all Prometheus targets that are down.

### Challenge 2

> Count all healthy Prometheus targets.

### Challenge 3

> Show Running Pods.

### Challenge 4

> Count Running Pods by namespace.

### Challenge 5

> Show memory usage grouped by Pod.

### Challenge 6

> Calculate CPU usage rate grouped by Pod.

---

# 🧠 Most Important Lesson 7 Concept

Don't think of PromQL as a collection of commands to memorize.

Think of it as:

```text
SELECT
   FILTER
   AGGREGATE
   CALCULATE
   GROUP
```

For example:

```promql
sum by (namespace) (
  rate(
    http_requests_total{
      environment="production"
    }[5m]
  )
)
```

Read it **inside → outside**:

```text
1. Find http_requests_total
          ↓
2. Only production
          ↓
3. Look at last 5 minutes
          ↓
4. Calculate rate
          ↓
5. Group by namespace
          ↓
6. Sum
```

That way of thinking will become very powerful when we build **Grafana dashboards and Prometheus alerts**.

---

## Next — Lesson 8: PromQL Deep Dive

We'll stay with PromQL for another lesson before moving heavily into Grafana.

We'll cover:

```text
Instant vectors
Range vectors
Label matching
sum / avg / max / min
sum by / sum without
rate vs irate
increase
topk
bottomk
absent
Operators
CPU queries
Memory queries
Pod restart queries
Node queries
```

And we'll start building a **real Kubernetes PromQL cheat sheet** that you can later reuse directly in Grafana dashboards and alert rules.
