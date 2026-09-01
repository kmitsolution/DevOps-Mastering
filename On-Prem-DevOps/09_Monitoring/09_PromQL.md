# Lesson 9 — Advanced PromQL + Kubernetes Filtering

Today we'll solve a common problem you may have already noticed:

```text
container_cpu_usage_seconds_total
```

can return many unexpected series:

```text
container=""
pod=""
namespace=""
```

or infrastructure/system containers.

The goal today is to learn **how to write clean Kubernetes PromQL queries** that you can later use directly in Grafana.

---

# 1. The Kubernetes Metric Problem

Run:

```promql
container_cpu_usage_seconds_total
```

You may get results like:

```text
container_cpu_usage_seconds_total{
    container="",
    namespace="",
    pod="",
    ...
}
```

and:

```text
container_cpu_usage_seconds_total{
    container="api",
    namespace="production",
    pod="api-7d8f9",
    ...
}
```

Why?

Because the metric may represent different types of container/cgroup data, including infrastructure-level series.

So we need to **filter the labels**.

---

# 2. Label Filtering

You already know:

```promql
metric{
    label="value"
}
```

For example:

```promql
container_cpu_usage_seconds_total{
    namespace="production"
}
```

This means:

> Only return metrics where namespace is `production`.

---

# 3. Filter Out Empty Labels

This is very useful.

Suppose you want only series that actually have a Pod:

```promql
container_cpu_usage_seconds_total{
    pod!=""
}
```

Meaning:

```text
pod is NOT empty
```

Similarly:

```promql
container_cpu_usage_seconds_total{
    container!=""
}
```

means:

> Only series with a container name.

---

# 4. Combining Filters

You can combine them:

```promql
container_cpu_usage_seconds_total{
    namespace!="",
    pod!="",
    container!=""
}
```

Now we're saying:

```text
namespace exists
AND
pod exists
AND
container exists
```

This can dramatically clean up Kubernetes queries.

---

# 5. CPU Usage by Pod — Clean Version

Instead of:

```promql
sum by (pod) (
  rate(container_cpu_usage_seconds_total[5m])
)
```

try:

```promql
sum by (namespace, pod) (
  rate(
    container_cpu_usage_seconds_total{
      namespace!="",
      pod!="",
      container!=""
    }[5m]
  )
)
```

Now you get something like:

```text
namespace       pod              CPU
--------------------------------------
production      api-1            0.25
production      api-2            0.40
production      frontend-1       0.15
monitoring      prometheus-0     1.20
```

Notice that we're grouping by:

```text
namespace
pod
```

rather than only:

```text
pod
```

This is safer because different namespaces can contain Pods with similar names.

---

# 6. Why `namespace + pod` Is Better Than Just `pod`

Imagine:

```text
production
  └── api

development
  └── api
```

If you use:

```promql
sum by (pod)
```

you might get:

```text
api → combined result
```

But:

```promql
sum by (namespace, pod)
```

keeps them separate:

```text
production/api
development/api
```

For Kubernetes dashboards, this is generally much more useful.

---

# 7. CPU by Namespace

Now we don't need the Pod dimension.

```promql
sum by (namespace) (
  rate(
    container_cpu_usage_seconds_total{
      namespace!="",
      pod!="",
      container!=""
    }[5m]
  )
)
```

Result:

```text
Namespace       CPU
--------------------
production      2.5
development     0.8
monitoring      3.2
```

---

# 8. Top 5 CPU-Consuming Pods

Combine `topk()` with the previous query:

```promql
topk(
  5,
  sum by (namespace, pod) (
    rate(
      container_cpu_usage_seconds_total{
        namespace!="",
        pod!="",
        container!=""
      }[5m]
    )
  )
)
```

This is a very useful Grafana panel.

---

# 9. Memory by Pod

Memory is a gauge, so don't use `rate()`.

Try:

```promql
sum by (namespace, pod) (
  container_memory_working_set_bytes{
    namespace!="",
    pod!="",
    container!=""
  }
)
```

You'll get:

```text
namespace       pod              memory
-----------------------------------------
production      api-1            450 MB
production      api-2            620 MB
production      frontend-1       300 MB
```

---

# 10. Top 5 Memory-Consuming Pods

```promql
topk(
  5,
  sum by (namespace, pod) (
    container_memory_working_set_bytes{
      namespace!="",
      pod!="",
      container!=""
    }
  )
)
```

This answers:

> Which five Pods are consuming the most memory right now?

Very useful during troubleshooting.

---

# 11. Namespace Memory

```promql
sum by (namespace) (
  container_memory_working_set_bytes{
    namespace!="",
    pod!="",
    container!=""
  }
)
```

Conceptually:

```text
production      4.2 GB
monitoring      2.8 GB
development     1.1 GB
```

---

# 12. Node CPU

Now let's move one level up.

A common approach is to use:

```promql
node_cpu_seconds_total
```

This is exposed by Node Exporter.

It's a counter.

So we use:

```promql
rate(node_cpu_seconds_total[5m])
```

But this metric contains multiple CPU modes:

```text
user
system
idle
iowait
...
```

So we need to think carefully about what we're calculating.

---

# 13. CPU Idle Percentage

One useful query is based on the `idle` mode.

Conceptually:

```promql
100 *
(
  1 -
  avg by (instance) (
    rate(node_cpu_seconds_total{
      mode="idle"
    }[5m])
  )
)
```

This gives an approximate CPU utilization percentage per instance.

The important concept is:

```text
CPU utilization
=
100% - CPU idle%
```

---

# 14. Why `instance`?

Node Exporter typically identifies a monitored node through labels such as:

```text
instance
job
```

For example:

```text
instance="192.168.1.10:9100"
```

So your result may look like:

```text
instance                    CPU %
----------------------------------
192.168.1.10:9100            35
192.168.1.11:9100            72
192.168.1.12:9100            48
```

Later, we'll make this prettier in Grafana by replacing technical instance names with useful node labels.

---

# 15. Node Memory Percentage

Node Exporter provides memory metrics such as:

```promql
node_memory_MemTotal_bytes
```

and:

```promql
node_memory_MemAvailable_bytes
```

Memory used can be conceptualized as:

```text
Total Memory - Available Memory
```

So:

```promql
(
  1 -
  node_memory_MemAvailable_bytes
  /
  node_memory_MemTotal_bytes
) * 100
```

This gives approximate memory utilization percentage.

---

# 16. Why This Query Is Powerful

Suppose your node has:

```text
Total     = 16 GB
Available = 4 GB
```

Then:

```text
Used ≈ 16 - 4
     ≈ 12 GB
```

Percentage:

```text
12 / 16 × 100
= 75%
```

PromQL performs the calculation automatically.

---

# 17. Pod Restart Monitoring

We previously used:

```promql
increase(
  kube_pod_container_status_restarts_total[1h]
)
```

Now let's make it cleaner.

```promql
sum by (namespace, pod) (
  increase(
    kube_pod_container_status_restarts_total[1h]
  )
) > 0
```

This asks:

> Which Pods had one or more container restarts during the last hour?

---

# 18. Top Restarting Pods

You can use:

```promql
topk(
  5,
  sum by (namespace, pod) (
    increase(
      kube_pod_container_status_restarts_total[1h]
    )
  )
)
```

You could get:

```text
Namespace       Pod             Restarts
-----------------------------------------
production      api-3              15
production      worker-2            8
development     api-1               4
```

Now you immediately know where to investigate.

---

# 19. Pod Availability

A useful Kubernetes metric is:

```promql
kube_deployment_status_replicas_available
```

and:

```promql
kube_deployment_spec_replicas
```

Suppose:

```text
Desired = 3
Available = 2
```

We have a problem.

You can compare them:

```promql
kube_deployment_status_replicas_available
<
kube_deployment_spec_replicas
```

This concept will become extremely useful when we build alerts.

---

# 20. Deployment Replica Difference

You can calculate:

```promql
kube_deployment_spec_replicas
-
kube_deployment_status_replicas_available
```

If:

```text
Desired = 5
Available = 3
```

result:

```text
2
```

Meaning:

> Two replicas are currently unavailable.

---

# 21. Filter by Namespace

Suppose you only care about production.

```promql
kube_deployment_spec_replicas{
  namespace="production"
}
```

Or:

```promql
kube_deployment_status_replicas_available{
  namespace="production"
}
```

This is the basic building block of environment-specific dashboards.

---

# 22. Regular Expression Matching

PromQL also supports regex.

Suppose you want:

```text
production
production-east
production-west
```

You could use:

```promql
metric{
  namespace=~"production.*"
}
```

The operator:

```text
=~
```

means:

> Regular expression match.

---

# 23. Exact vs Regex

Remember:

```text
=~
```

means regex matching.

While:

```text
=
```

means exact matching.

Example:

```promql
namespace="production"
```

matches only:

```text
production
```

while:

```promql
namespace=~"production.*"
```

can match:

```text
production
production-east
production-west
```

---

# 24. Negative Regex

You can also use:

```text
!~
```

For example:

```promql
namespace!~"kube-.*"
```

means:

> Exclude namespaces beginning with `kube-`.

This can be useful for creating application-focused dashboards.

---

# 25. A Very Useful Kubernetes Pattern

You'll frequently see queries structured like this:

```promql
sum by (namespace, pod) (
  rate(
    metric{
      namespace!="",
      pod!="",
      container!=""
    }[5m]
  )
)
```

Break it down:

```text
Metric
  ↓
Filter unwanted series
  ↓
Select time range
  ↓
rate()
  ↓
Group by namespace + pod
  ↓
sum()
```

This is a pattern worth remembering.

---

# 26. PromQL Query Development Method

When creating a new query, don't write the entire thing immediately.

Use this process.

### Step 1 — Find metric

```promql
container_cpu_usage_seconds_total
```

### Step 2 — Inspect labels

Look at:

```text
namespace
pod
container
node
```

### Step 3 — Filter

```promql
{
  namespace!="",
  pod!="",
  container!=""
}
```

### Step 4 — Determine metric type

CPU:

```text
Counter
```

Memory:

```text
Gauge
```

### Step 5 — Apply function

CPU:

```promql
rate(...[5m])
```

Memory:

```text
direct value
```

### Step 6 — Aggregate

```promql
sum by (namespace, pod)
```

### Step 7 — Rank if necessary

```promql
topk(5, ...)
```

This approach will save you a lot of frustration.

---

# 27. Your Kubernetes PromQL Toolkit

At this point, you should know these patterns:

### Target health

```promql
up
```

### Down targets

```promql
up == 0
```

### CPU

```promql
rate(container_cpu_usage_seconds_total[5m])
```

### CPU by Pod

```promql
sum by (namespace, pod) (
  rate(container_cpu_usage_seconds_total[5m])
)
```

### Memory

```promql
container_memory_working_set_bytes
```

### Memory by Pod

```promql
sum by (namespace, pod) (
  container_memory_working_set_bytes
)
```

### Restarts

```promql
increase(kube_pod_container_status_restarts_total[1h])
```

### Top N

```promql
topk(5, metric)
```

### Group

```promql
sum by (namespace) (metric)
```

---

# 🧪 Lesson 9 Practical Lab

Open Prometheus:

```text
http://localhost:9090
```

Now try these.

### Exercise 1 — CPU by Pod

```promql
sum by (namespace, pod) (
  rate(
    container_cpu_usage_seconds_total{
      namespace!="",
      pod!="",
      container!=""
    }[5m]
  )
)
```

---

### Exercise 2 — Top 5 CPU Pods

```promql
topk(
  5,
  sum by (namespace, pod) (
    rate(
      container_cpu_usage_seconds_total{
        namespace!="",
        pod!="",
        container!=""
      }[5m]
    )
  )
)
```

---

### Exercise 3 — Memory by Pod

```promql
sum by (namespace, pod) (
  container_memory_working_set_bytes{
    namespace!="",
    pod!="",
    container!=""
  }
)
```

---

### Exercise 4 — Top 5 Memory Pods

```promql
topk(
  5,
  sum by (namespace, pod) (
    container_memory_working_set_bytes{
      namespace!="",
      pod!="",
      container!=""
    }
  )
)
```

---

### Exercise 5 — Restarting Pods

```promql
sum by (namespace, pod) (
  increase(
    kube_pod_container_status_restarts_total[1h]
  )
) > 0
```

---

### Exercise 6 — Deployment Availability

First:

```promql
kube_deployment_spec_replicas
```

Then:

```promql
kube_deployment_status_replicas_available
```

Then compare:

```promql
kube_deployment_status_replicas_available
<
kube_deployment_spec_replicas
```

---

# 🎯 Challenge

Try writing these yourself.

### Challenge 1

Find the **top 3 Pods by memory usage**.

### Challenge 2

Find the **top 5 namespaces by CPU usage**.

### Challenge 3

Find Pods that had **more than 3 restarts in the last hour**.

### Challenge 4

Find Deployments where:

```text
Available replicas < Desired replicas
```

### Challenge 5

Calculate **node memory utilization percentage**.

---

# 🧠 One Important Rule

When working with Kubernetes metrics, **never blindly copy a PromQL query from the internet and assume it is correct for your cluster**.

First inspect:

```promql
metric_name
```

Then inspect its labels.

For example:

```text
namespace
pod
container
node
instance
job
```

Then build the query.

Different Kubernetes versions, container runtimes, exporters, and monitoring-stack versions can expose slightly different metric labels.

---

# 🏁 Where We Are Now

You've now covered:

```text
Lesson 1
Monitoring Fundamentals
        ↓
Lesson 2
Prometheus Architecture
        ↓
Lesson 3
Install kube-prometheus-stack
        ↓
Lesson 4
Prometheus Components
        ↓
Lesson 5
Targets & Scraping
        ↓
Lesson 6
Metrics, Labels & Time Series
        ↓
Lesson 7
PromQL Fundamentals
        ↓
Lesson 8
PromQL Deep Dive
        ↓
Lesson 9
Kubernetes Filtering
```

The next major stage is **Grafana**.

# Lesson 10 — Grafana Fundamentals

We'll connect:

```text
Grafana
    │
    ▼
Prometheus
    │
    ▼
PromQL
```

and build your **first Kubernetes dashboard** from scratch.

We'll cover:

* Grafana login
* Prometheus datasource
* Explore
* Panels
* Queries
* Time ranges
* Table vs Gauge vs Stat vs Time Series
* First CPU panel
* First memory panel
* First Pod count panel
* First dashboard

Then we'll gradually turn it into a **production-style Kubernetes monitoring dashboard**.
