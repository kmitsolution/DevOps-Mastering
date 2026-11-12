# Lesson 1 — Monitoring & Observability Fundamentals

Since your final target is **Kubernetes + Prometheus + Grafana + GitLab CI/CD**, we'll learn monitoring from a DevOps perspective.

---

## 1. What is Monitoring?

**Monitoring means continuously collecting information about a system so that we can know whether it is healthy or having problems.**

Imagine you have this application:

```text
User
  ↓
Load Balancer
  ↓
Frontend
  ↓
Backend API
  ↓
PostgreSQL
```

You need to know:

* Is the application running?
* How much CPU is being used?
* How much memory is being used?
* How many users are accessing it?
* How many requests are failing?
* How long are requests taking?
* Are containers restarting?
* Is the database available?

Monitoring gives you answers to these questions.

---

# 2. Why Do We Need Monitoring?

Suppose your Kubernetes application suddenly becomes slow.

Without monitoring:

```text
User
  ↓
"Application is slow!"
  ↓
DevOps Engineer
  ↓
????
```

You start manually checking:

```bash
kubectl get pods
kubectl get nodes
kubectl describe pod
kubectl logs
top
```

This can take a lot of time.

With monitoring:

```text
User
  ↓
Application becomes slow
  ↓
Prometheus detects high latency
  ↓
Grafana shows the problem
  ↓
Alertmanager sends notification
  ↓
DevOps Engineer investigates
```

That's the goal.

---

# 3. Three Important Observability Signals

You will hear these three terms repeatedly:

```text
             Observability
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
    Metrics      Logs      Traces
```

## Metrics

Metrics are **numbers measured over time**.

Examples:

```text
CPU = 75%
Memory = 4 GB
Requests = 1,200/sec
Errors = 20/sec
Latency = 250 ms
```

Prometheus specializes in **metrics**.

---

## Logs

Logs tell you what happened.

Example:

```text
2026-08-31 11:20:15 ERROR
Database connection failed
```

Kubernetes example:

```bash
kubectl logs my-pod
```

Logs answer:

> **What happened?**

---

## Traces

Traces follow a request through multiple services.

For example:

```text
User
 │
 ▼
Frontend
 │
 ▼
API
 │
 ├── Authentication
 │
 ├── Order Service
 │
 └── PostgreSQL
```

A trace can tell you:

```text
Request
  │
  ├── Frontend       20 ms
  ├── API            50 ms
  ├── Order Service  100 ms
  └── Database       500 ms  ← Problem
```

Traces answer:

> **Where did the request spend its time?**

We'll focus primarily on **metrics** in this course.

---

# 4. What is a Metric?

A metric is basically a measurement.

For example, suppose your application receives requests:

```text
10:00 → 100 requests
10:01 → 150 requests
10:02 → 200 requests
10:03 → 300 requests
```

Prometheus can store these measurements over time.

Conceptually:

```text
Requests
  │
300│             ●
  │
200│        ●
  │
150│    ●
  │
100│ ●
  └──────────────────
    10:00 10:01 10:02 10:03
```

This is where Grafana becomes useful.

Prometheus **stores and queries** the metrics.

Grafana **visualizes** them.

---

# 5. Prometheus vs Grafana

This distinction is extremely important.

### Prometheus

Think:

> **Collect + Store + Query metrics**

```text
Application
     ↓
  Metrics
     ↓
 Prometheus
     ↓
 Time Series Database
```

### Grafana

Think:

> **Visualize metrics**

```text
Prometheus
     ↓
    Data
     ↓
  Grafana
     ↓
Dashboard
```

So:

```text
Prometheus = Monitoring / Metrics Engine

Grafana = Visualization / Dashboard
```

They work together but they are **not the same thing**.

---

# 6. Simple Example

Suppose we have a Kubernetes pod:

```text
my-api-pod
```

It is consuming:

```text
CPU     = 70%
Memory  = 500 MB
Requests = 200/sec
Errors   = 5/sec
```

Prometheus collects these metrics.

Grafana can display:

```text
┌──────────────────────────────────┐
│       Kubernetes Dashboard       │
├──────────────────────────────────┤
│ CPU             70%              │
│ ███████████████░░░░              │
│                                  │
│ Memory          500 MB           │
│ ██████████░░░░░░░░               │
│                                  │
│ Requests        200/sec          │
│                                  │
│ Errors          5/sec            │
└──────────────────────────────────┘
```

---

# 7. Kubernetes Monitoring

Now let's connect this to Kubernetes.

Imagine:

```text
Kubernetes Cluster
│
├── Node 1
│   ├── Pod A
│   ├── Pod B
│   └── Pod C
│
├── Node 2
│   ├── Pod D
│   └── Pod E
│
└── Node 3
    ├── Pod F
    └── Pod G
```

You want to monitor:

### Cluster

```text
Cluster CPU
Cluster Memory
Number of Nodes
Number of Pods
```

### Nodes

```text
Node CPU
Node Memory
Disk
Network
Node availability
```

### Pods

```text
Pod CPU
Pod Memory
Pod status
Pod restarts
```

### Applications

```text
Requests
Errors
Latency
Availability
```

Prometheus becomes the central metrics system.

---

# 8. How Does Prometheus Get Metrics?

This is one of the most important concepts for Lesson 1.

Prometheus generally uses a **pull model**.

```text
                Prometheus
                    │
                    │ HTTP request
                    │ "Give me metrics"
                    ↓
             ┌─────────────┐
             │ Application │
             │ /metrics    │
             └─────────────┘
                    │
                    │
                    │ metrics
                    ▼
                Prometheus
```

For example, an application may expose:

```text
http://application:8080/metrics
```

and return:

```text
http_requests_total 1500
http_errors_total 25
```

Prometheus periodically scrapes that endpoint.

---

# 9. What is an Exporter?

Sometimes an application or system doesn't expose Prometheus metrics directly.

An **exporter** converts information into Prometheus metrics.

Example:

```text
Linux Server
     │
     ▼
Node Exporter
     │
     ▼
Prometheus
```

Node Exporter can expose things such as:

```text
CPU
Memory
Disk
Network
```

So:

```text
System → Exporter → Prometheus
```

You'll learn exporters properly in a later lesson.

---

# 10. Kubernetes Monitoring Architecture

Eventually our architecture will look approximately like:

```text
                         ┌───────────────┐
                         │    Grafana    │
                         └───────▲───────┘
                                 │
                              PromQL
                                 │
                         ┌───────┴───────┐
                         │  Prometheus   │
                         └───────▲───────┘
                                 │
                         Scrape Metrics
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
              ▼                  ▼                  ▼
        Node Exporter     kube-state-metrics   Application
              │                  │                  │
              ▼                  ▼                  ▼
           Nodes             Kubernetes          /metrics
```

Later we'll add:

```text
Prometheus
    │
    ▼
Alertmanager
    │
    ├── Email
    ├── Teams
    └── Slack
```

And finally:

```text
GitLab CI/CD
      │
      ▼
 Kubernetes
      │
      ▼
 Prometheus
      │
      ▼
 Grafana
      │
      ▼
 Alerts
```

---

# 11. Monitoring vs Alerting

These are different.

### Monitoring

You observe:

```text
CPU = 90%
```

### Alerting

You define a rule:

```text
IF CPU > 80%
FOR 5 minutes
THEN alert
```

So:

```text
Metric
  ↓
Prometheus
  ↓
Alert Rule
  ↓
Alert
  ↓
Alertmanager
  ↓
Notification
```

We'll spend several lessons on this.

---

# 12. A Real Kubernetes Scenario

Imagine your application has 3 replicas:

```yaml
replicas: 3
```

Kubernetes:

```text
Pod 1 → Running
Pod 2 → Running
Pod 3 → Running
```

Everything looks healthy.

Then:

```text
Pod 1 → Running
Pod 2 → CrashLoopBackOff
Pod 3 → Running
```

Kubernetes knows about the failure.

But your monitoring system can provide more information:

```text
Pod restarts       ↑
CPU                 ↑
Memory              ↑
HTTP errors         ↑
Latency             ↑
Available replicas  ↓
```

Grafana can show all of this visually.

Prometheus can detect:

```text
available replicas < desired replicas
```

and generate an alert.

---

# 13. The Mental Model You Should Remember

For now, remember this:

```text
             SYSTEM
                │
                ▼
             Metrics
                │
                ▼
           Prometheus
                │
             PromQL
                │
                ▼
             Grafana
                │
                ▼
           Visualization
```

And for alerting:

```text
             Metrics
                │
                ▼
           Prometheus
                │
           Alert Rule
                │
                ▼
          Alertmanager
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
     Email    Teams    Slack
```

---

# 🧪 Lesson 1 Exercise

Before moving to Lesson 2, make sure you can answer these.

### Q1

What is the primary purpose of Prometheus?

A. Container orchestration
B. Metrics collection, storage and querying
C. Creating Docker images
D. Kubernetes deployment

### Q2

What is the primary purpose of Grafana?

A. Container runtime
B. Kubernetes scheduler
C. Metrics visualization and dashboards
D. Git repository

### Q3

Which one represents a metric?

A.

```text
Database connection failed
```

B.

```text
CPU usage = 82%
```

C.

```text
kubectl get pods
```

D.

```text
Deployment YAML
```

### Q4

What is the difference between:

```text
Prometheus
```

and

```text
Grafana
```

### Q5

In a Kubernetes environment, name **at least 5 things** you would want to monitor.

---

## 🎯 Lesson 1 Target

After this lesson, you should have this mental picture:

```text
                 ┌─────────────────────┐
                 │     Kubernetes      │
                 │                     │
                 │ Nodes               │
                 │ Pods                │
                 │ Applications        │
                 └──────────┬──────────┘
                            │
                         Metrics
                            │
                            ▼
                    ┌───────────────┐
                    │  Prometheus   │
                    │               │
                    │ Collect       │
                    │ Store         │
                    │ Query         │
                    └───────┬───────┘
                            │
                          PromQL
                            │
                            ▼
                    ┌───────────────┐
                    │    Grafana    │
                    │               │
                    │ Dashboards    │
                    │ Visualization │
                    └───────────────┘
```

**Next: Lesson 2 — Prometheus Architecture.** We'll understand **Prometheus Server, TSDB, scrape targets, exporters, `/metrics`, labels, pull model, and the complete request/data flow**, before installing anything.
