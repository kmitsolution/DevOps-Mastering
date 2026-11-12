# Lesson 2 — Prometheus Architecture

In Lesson 1, we established:

```text
Kubernetes
    ↓
Metrics
    ↓
Prometheus
    ↓
PromQL
    ↓
Grafana
```

Now let's understand **what actually happens inside Prometheus**.

---

# 1. Prometheus Architecture — Big Picture

The architecture you should remember is:

```text
                    ┌─────────────────────┐
                    │      Grafana        │
                    │   Visualization     │
                    └──────────┬──────────┘
                               │
                            PromQL
                               │
                               ▼
                    ┌─────────────────────┐
                    │     Prometheus      │
                    │                     │
                    │  Query Engine       │
                    │  TSDB               │
                    │  Scraper            │
                    │  Rule Engine        │
                    └──────────┬──────────┘
                               │
                         HTTP /metrics
                               │
             ┌─────────────────┼─────────────────┐
             │                 │                 │
             ▼                 ▼                 ▼
       Node Exporter    kube-state-metrics   Application
             │                 │                 │
             ▼                 ▼                 ▼
          Node OS          Kubernetes         /metrics
```

There are several important components here.

---

# 2. Prometheus Is a Time-Series Database

This is the first major concept.

Prometheus doesn't simply collect:

```text
CPU = 70%
```

It stores the value **along with time**.

For example:

```text
Time                 CPU
────────────────────────────
10:00                40%
10:01                45%
10:02                60%
10:03                75%
10:04                82%
```

This is called **time-series data**.

That's why you can ask Grafana:

> Show me CPU usage for the last 6 hours.

Prometheus has historical samples to answer that query.

---

# 3. TSDB

Prometheus has its own **Time Series Database (TSDB)**.

Conceptually:

```text
Metric
   +
Labels
   +
Timestamp
   +
Value
   ↓
Time Series
```

For example:

```text
http_requests_total{
    method="GET",
    service="frontend"
}
```

could have:

```text
Timestamp             Value
────────────────────────────
10:00                  100
10:01                  120
10:02                  150
10:03                  180
```

The combination of:

```text
metric + labels
```

identifies a particular time series.

We'll spend an entire lesson on labels because they are **extremely important in Kubernetes monitoring**.

---

# 4. What Is Scraping?

Prometheus uses a **pull model**.

Prometheus periodically asks targets:

> Give me your current metrics.

Example:

```text
Prometheus
    │
    │ HTTP GET
    │ /metrics
    ▼
Application
    │
    │ metrics
    ▼
Prometheus
```

For example:

```text
GET http://10.0.1.10:8080/metrics
```

The application responds:

```text
http_requests_total 1500
http_errors_total 20
```

Prometheus stores these values.

---

# 5. What Is a Target?

A **target** is something Prometheus monitors.

For example:

```text
Targets
│
├── node1:9100
├── node2:9100
├── node3:9100
└── application:8080
```

Prometheus periodically scrapes each target.

For example:

```text
Every 15 seconds

Prometheus
    │
    ├──→ node1:9100/metrics
    │
    ├──→ node2:9100/metrics
    │
    ├──→ node3:9100/metrics
    │
    └──→ application:8080/metrics
```

---

# 6. What Is an Exporter?

An exporter is a component that exposes metrics in a format Prometheus understands.

A common example is **Node Exporter**.

Imagine a Linux server:

```text
Linux Server
│
├── CPU
├── Memory
├── Disk
└── Network
```

Node Exporter collects information from the OS:

```text
Linux Server
     │
     ▼
Node Exporter
     │
     ▼
/metrics
     │
     ▼
Prometheus
```

Node Exporter might expose metrics such as:

```text
node_cpu_seconds_total
node_memory_MemAvailable_bytes
node_filesystem_avail_bytes
```

---

# 7. Application Metrics

Your application doesn't necessarily need an exporter.

A modern application can expose:

```text
/metrics
```

For example:

```text
http://my-api:8080/metrics
```

Response:

```text
http_requests_total 15000
http_errors_total 200
```

Architecture:

```text
Application
     │
     └── /metrics
             │
             ▼
        Prometheus
```

Later, we'll build an application that exposes its own metrics.

---

# 8. Kubernetes Changes Everything

In a normal VM environment you might have:

```text
Prometheus
    │
    ├── server1
    ├── server2
    └── server3
```

But Kubernetes is dynamic.

Pods are constantly created and destroyed.

For example:

```text
10:00

Pod A → 10.1.0.5
Pod B → 10.1.0.6
Pod C → 10.1.0.7
```

Later:

```text
10:10

Pod A → deleted
Pod D → created
Pod E → created
```

Their IP addresses can change.

You don't want to manually configure Prometheus every time.

That's why **service discovery** becomes important.

---

# 9. Service Discovery

Prometheus can discover Kubernetes targets dynamically.

Conceptually:

```text
                 Kubernetes API
                       │
                       ▼
                Service Discovery
                       │
             ┌─────────┼─────────┐
             ▼         ▼         ▼
           Pod A      Pod B     Pod C
             │         │         │
             └─────────┼─────────┘
                       ▼
                  Prometheus
```

Instead of saying:

```text
Monitor 10.1.0.5
Monitor 10.1.0.6
Monitor 10.1.0.7
```

Prometheus can say:

> Discover the Kubernetes objects matching my monitoring configuration.

This becomes extremely important later with:

```text
ServiceMonitor
PodMonitor
Prometheus Operator
```

---

# 10. kube-state-metrics

Here's another important Kubernetes component.

Kubernetes itself has lots of state information:

```text
Deployment desired replicas = 3
Available replicas = 2

Pod status = Running

DaemonSet desired = 5
DaemonSet available = 4
```

**kube-state-metrics** exposes Kubernetes object state as Prometheus metrics.

Architecture:

```text
             Kubernetes API
                    │
                    ▼
           kube-state-metrics
                    │
                 /metrics
                    │
                    ▼
               Prometheus
```

For example:

```text
kube_deployment_spec_replicas
kube_deployment_status_replicas_available
kube_pod_status_phase
```

This lets Prometheus answer questions like:

> How many replicas does my deployment want?

and:

> How many replicas are actually available?

---

# 11. Node Metrics vs Kubernetes State

This distinction is very important.

### Node Exporter

Answers questions about the **machine**:

```text
CPU
Memory
Disk
Network
Filesystem
```

### kube-state-metrics

Answers questions about **Kubernetes objects**:

```text
Pods
Deployments
Services
DaemonSets
StatefulSets
Jobs
```

Think:

```text
                    Monitoring
                        │
             ┌──────────┴──────────┐
             │                     │
        Infrastructure          Kubernetes
             │                     │
             ▼                     ▼
       Node Exporter        kube-state-metrics
             │                     │
             └──────────┬──────────┘
                        ▼
                   Prometheus
```

---

# 12. Prometheus Query Engine

Prometheus doesn't only store metrics.

It can **query** them using PromQL.

For example:

```promql
up
```

asks:

> Which monitored targets are currently up?

Another example:

```promql
node_memory_MemAvailable_bytes
```

asks:

> Give me available memory.

Later you'll write more sophisticated queries:

```promql
rate(http_requests_total[5m])
```

Meaning approximately:

> Calculate the request rate over the last 5 minutes.

Grafana sends these PromQL queries to Prometheus.

---

# 13. Grafana's Role

Grafana doesn't normally collect the metrics itself.

Instead:

```text
User
  │
  ▼
Grafana
  │
  │ PromQL
  ▼
Prometheus
  │
  ▼
Metrics
```

Example:

You open Grafana and select:

```text
Last 1 hour
```

Grafana sends a query to Prometheus.

Prometheus returns the data.

Grafana turns it into:

```text
CPU Usage
100% ┤
 80% ┤                ●
 60% ┤          ●─────●
 40% ┤    ●─────●
 20% ┤●───●
     └────────────────────
       10  20  30  40  50
```

---

# 14. Prometheus Configuration

Prometheus has a configuration file commonly called:

```text
prometheus.yml
```

A simplified example:

```yaml
global:
  scrape_interval: 15s

scrape_configs:

  - job_name: "prometheus"
    static_configs:
      - targets:
          - "localhost:9090"
```

The important part is:

```yaml
scrape_interval: 15s
```

It means Prometheus attempts to scrape targets every 15 seconds.

And:

```yaml
job_name: "prometheus"
```

identifies the monitoring job.

We'll create this configuration ourselves in the next lessons.

---

# 15. What Happens During a Scrape?

Let's follow one complete cycle.

Suppose:

```text
Prometheus
```

is configured to monitor:

```text
node1:9100
```

### Step 1

Prometheus discovers the target.

```text
node1:9100
```

### Step 2

Prometheus sends:

```text
GET /metrics
```

### Step 3

Node Exporter responds:

```text
node_cpu_seconds_total ...
node_memory_MemAvailable_bytes ...
node_filesystem_avail_bytes ...
```

### Step 4

Prometheus parses the metrics.

### Step 5

Prometheus stores them in its TSDB.

```text
Metric + Labels + Timestamp + Value
```

### Step 6

You query them using PromQL.

### Step 7

Grafana visualizes them.

Complete flow:

```text
Node
 │
 ▼
Node Exporter
 │
 │ /metrics
 ▼
Prometheus
 │
 ├── Scraper
 │
 ├── TSDB
 │
 └── Query Engine
 │
 │ PromQL
 ▼
Grafana
 │
 ▼
Dashboard
```

---

# 16. Where Does Alerting Fit?

We haven't studied Alertmanager yet, but understand the architecture now.

```text
                    Prometheus
                        │
             ┌──────────┴──────────┐
             │                     │
          Queries               Alert Rules
             │                     │
             ▼                     ▼
          Grafana              Alertmanager
                                   │
                         ┌─────────┼─────────┐
                         ▼         ▼         ▼
                       Email     Slack     Teams
```

Prometheus evaluates alert rules.

Alertmanager handles the notification workflow.

We'll study this in depth later.

---

# 17. Full Kubernetes Architecture

Here's the architecture I want you to eventually be comfortable explaining in an interview:

```text
                         ┌───────────────┐
                         │    Grafana    │
                         │               │
                         │ Dashboards    │
                         │ Visualization │
                         └───────▲───────┘
                                 │
                               PromQL
                                 │
                                 ▼
                      ┌─────────────────────┐
                      │     Prometheus      │
                      │                     │
                      │ Scraper             │
                      │ TSDB                │
                      │ Query Engine        │
                      │ Rule Engine         │
                      └──────────┬──────────┘
                                 │
                         Service Discovery
                                 │
                 ┌───────────────┼────────────────┐
                 │               │                │
                 ▼               ▼                ▼
          Node Exporter   kube-state-metrics   Application
                 │               │                │
                 │               │                │
                 ▼               ▼                ▼
               Nodes       Kubernetes API      /metrics
                 │               │                │
                 └───────────────┼────────────────┘
                                 │
                                 ▼
                           Metrics Data
```

Later:

```text
                      Prometheus
                           │
                           ▼
                      Alert Rules
                           │
                           ▼
                     Alertmanager
                           │
                 ┌─────────┼─────────┐
                 ▼         ▼         ▼
               Email     Teams     Slack
```

---

# 18. One Important Interview Question

### Why can't we just use `kubectl` for monitoring?

`kubectl` is primarily a **Kubernetes management and inspection CLI**.

For example:

```bash
kubectl get pods
```

gives you a current snapshot.

But monitoring requires:

```text
Historical data
      +
Time-series metrics
      +
Queries
      +
Dashboards
      +
Alerts
```

For example, you might want to know:

> What was the CPU utilization of this pod during the last 24 hours?

That's where Prometheus + Grafana become useful.

---

# 19. Another Important Question

### Why do we need both Prometheus and Grafana?

Because they solve different problems.

```text
Prometheus
──────────
Collect
Store
Query
Alert
```

while:

```text
Grafana
───────
Visualize
Dashboard
Explore
Correlate data
```

So:

```text
Prometheus = Metrics engine

Grafana = Visualization platform
```

---

# 🧪 Lesson 2 Exercise

Try answering these without looking back.

### Q1

What is a Prometheus target?

### Q2

What does Prometheus scrape?

### Q3

What is the purpose of `/metrics`?

### Q4

What does Node Exporter provide?

### Q5

What does kube-state-metrics provide?

### Q6

What is the difference between:

```text
Node Exporter
```

and

```text
kube-state-metrics
```

### Q7

Why is Kubernetes service discovery important?

### Q8

What is Prometheus TSDB?

### Q9

What language do we use to query Prometheus?

### Q10

Explain this flow in your own words:

```text
Kubernetes
   ↓
Exporter / Application
   ↓
/metrics
   ↓
Prometheus
   ↓
PromQL
   ↓
Grafana
```

---

## 🔑 Lesson 2 — Remember These 8 Terms

If you understand these, you're ready for the next lesson:

```text
1. Target
2. Scrape
3. /metrics
4. Exporter
5. Service Discovery
6. TSDB
7. PromQL
8. Grafana
```

### Next lesson

**Lesson 3 — Install Prometheus with Docker**

We'll do this hands-on:

```text
Windows/Linux
     ↓
Docker
     ↓
Prometheus
     ↓
prometheus.yml
     ↓
localhost:9090
     ↓
First metric
     ↓
First PromQL query
```

Then we'll install **Grafana** and connect it to Prometheus.
