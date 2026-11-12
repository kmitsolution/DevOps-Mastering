# Phase 1 — Monitoring Fundamentals

Before touching Prometheus, understand these concepts:

### 1. What is monitoring?

For example:

```text
Application
     ↓
   Metrics
     ↓
 Prometheus
     ↓
   Grafana
     ↓
 Dashboard
```

You should understand:

* CPU utilization
* Memory utilization
* Request count
* Error count
* Request latency
* Disk usage
* Network traffic
* Pod restarts
* Pod availability

### 2. Metrics vs Logs vs Traces

Understand:

```text
Metrics → What is happening?
Logs    → What happened?
Traces  → Where did the request go?
```

For example:

```text
HTTP Requests
      │
      ├── Metrics → 10,000 requests/min
      │
      ├── Logs → "Database connection failed"
      │
      └── Trace → Client → API → Service → PostgreSQL
```

This becomes important later when you learn full **observability**.

---

# Phase 2 — Prometheus Fundamentals

Start with Prometheus running in Docker.

Learn:

### Prometheus architecture

```text
                  ┌───────────────┐
                  │   Grafana     │
                  └───────┬───────┘
                          │
                       PromQL
                          │
                          ▼
                  ┌───────────────┐
                  │  Prometheus   │
                  └───────┬───────┘
                          │
                    Scrape metrics
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
     Node Exporter   App Metrics     Kubernetes
```

Learn these terms:

* Prometheus server
* Target
* Scraping
* Exporter
* Time series
* Labels
* Metric
* Prometheus configuration
* `/metrics`
* Pull model

---

# Phase 3 — Metrics

This is extremely important.

Learn:

### Counter

Something that continuously increases.

```text
http_requests_total
```

Example:

```text
100
101
102
103
...
```

### Gauge

A value that can increase or decrease.

```text
node_memory_available_bytes
```

Example:

```text
8 GB
6 GB
4 GB
7 GB
```

### Histogram

Used for distributions such as request latency.

```text
http_request_duration_seconds
```

You'll eventually write queries such as:

```promql
rate(http_requests_total[5m])
```

and:

```promql
histogram_quantile(
  0.95,
  rate(http_request_duration_seconds_bucket[5m])
)
```

Don't worry about mastering these immediately.

---

# Phase 4 — PromQL

This should be one of your strongest areas.

Learn PromQL progressively:

### Level 1

```promql
up
```

```promql
process_cpu_seconds_total
```

### Level 2 — filtering

```promql
up{job="node"}
```

### Level 3 — aggregation

```promql
sum(up)
```

```promql
sum by (instance) (up)
```

### Level 4 — rates

```promql
rate(http_requests_total[5m])
```

### Level 5 — Kubernetes queries

Eventually:

```promql
sum by (namespace) (
  kube_pod_container_resource_requests
)
```

and:

```promql
sum by (pod) (
  rate(container_cpu_usage_seconds_total[5m])
)
```

This is where Prometheus becomes really powerful.

---

# Phase 5 — Grafana

Once PromQL is comfortable, start Grafana.

Learn:

```text
Grafana
  │
  ├── Datasource
  │       └── Prometheus
  │
  ├── Dashboard
  │
  ├── Panel
  │
  ├── Query
  │
  ├── Variables
  │
  └── Alerts
```

Build dashboards yourself rather than immediately importing dashboards.

Start with:

### Dashboard 1 — Linux

```text
CPU
Memory
Disk
Network
Load
```

Then:

### Dashboard 2 — Prometheus

```text
Targets
Scrape duration
Samples
Memory
Queries
```

Then:

### Dashboard 3 — Kubernetes

```text
Nodes
Namespaces
Pods
CPU
Memory
Restarts
Network
```

---

# Phase 6 — Kubernetes Monitoring

This is where your learning becomes really useful.

You should understand:

```text
Kubernetes Cluster
│
├── Control Plane
│
├── Worker Node
│
│    ├── kubelet
│    ├── container runtime
│    └── Pods
│
└── Applications
```

Then introduce:

### kube-state-metrics

It exposes Kubernetes object information such as:

```text
Pods
Deployments
DaemonSets
StatefulSets
Nodes
Namespaces
Jobs
```

For example:

```promql
kube_pod_status_phase
```

or:

```promql
kube_deployment_status_replicas_available
```

---

# Phase 7 — Prometheus Operator

This is **very important for Kubernetes**.

Instead of manually maintaining a huge Prometheus configuration, learn the Kubernetes-native approach.

You'll encounter:

```text
Prometheus Operator
        │
        ├── Prometheus
        ├── Alertmanager
        ├── ServiceMonitor
        ├── PodMonitor
        └── PrometheusRule
```

For example:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
```

This is the direction I recommend you take for real Kubernetes environments.

---

# Phase 8 — Alerting

Now move from:

> "I can see something is wrong."

to:

> "The system tells me something is wrong."

Example:

```text
Pod CPU > 80%
        ↓
Prometheus evaluates rule
        ↓
Alert fires
        ↓
Alertmanager
        ↓
Email / Teams / Slack
```

Learn alert concepts:

* Alert expression
* Threshold
* `for`
* Pending
* Firing
* Labels
* Annotations
* Severity

Example:

```yaml
alert: HighCPUUsage
expr: cpu_usage > 80
for: 5m
```

---

# Phase 9 — Alertmanager

Don't skip this.

Prometheus detects the problem.

**Alertmanager manages the alert.**

Learn:

```text
Prometheus
     │
     │ Alert
     ▼
Alertmanager
     │
     ├── Grouping
     ├── Routing
     ├── Silencing
     ├── Inhibition
     │
     └── Receivers
           ├── Email
           ├── Slack
           └── Teams
```

Example:

```text
Production alert
      ↓
severity=critical
      ↓
Production route
      ↓
DevOps Team
```

while:

```text
severity=warning
      ↓
Warning route
      ↓
Email
```

---

# Phase 10 — Grafana Alerting

Then learn Grafana's own alerting capabilities.

Understand the difference between:

```text
Prometheus Alerting
        +
Alertmanager
```

and:

```text
Grafana Alerting
```

Learn:

* Alert rules
* Contact points
* Notification policies
* Alert groups
* Silence
* Alert history

---

# Phase 11 — Monitor Your Own Application

This is an important milestone.

Instead of monitoring only Kubernetes infrastructure:

```text
Kubernetes
     ↓
Your Application
```

Expose application metrics.

For example:

```text
/app
/health
/metrics
```

Your application might expose:

```text
http_requests_total
http_request_duration_seconds
database_connections
```

Then:

```text
Application
      ↓
/metrics
      ↓
Prometheus
      ↓
Grafana
```

Now you're doing **real application monitoring**.

---

# Phase 12 — Production Kubernetes Dashboard

Build a complete dashboard containing:

### Cluster

```text
Cluster CPU
Cluster Memory
Node count
Pod count
```

### Nodes

```text
CPU
Memory
Disk
Network
Node availability
```

### Namespace

```text
CPU
Memory
Pod count
Restarts
```

### Pods

```text
CPU
Memory
Restart count
Status
```

### Application

```text
Requests/sec
Errors/sec
Latency
Availability
```

---

# Phase 13 — GitLab CI/CD Integration

This is where your previous GitLab/Kubernetes learning connects nicely.

Your final architecture should look like:

```text
              Developer
                  │
                  ▼
              GitLab
                  │
                  ▼
            GitLab CI/CD
                  │
        ┌─────────┴─────────┐
        │                   │
     Build               Test
        │                   │
        └─────────┬─────────┘
                  │
                  ▼
              Docker Image
                  │
                  ▼
              Registry
                  │
                  ▼
             Kubernetes
                  │
                  ▼
             Application
                  │
             /metrics
                  │
                  ▼
             Prometheus
                  │
                  ▼
               Grafana
                  │
                  ▼
              Dashboard
```

Then add:

```text
Prometheus
     │
     ▼
Alertmanager
     │
     ├── Email
     ├── Slack
     └── Teams
```

---

# Phase 14 — The Final Project

I recommend that we eventually build **one complete project** rather than doing disconnected examples.

### Project: Kubernetes Application Monitoring with GitLab CI/CD

We'll create:

```text
GitLab Repository
│
├── application/
│
├── Dockerfile
│
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── servicemonitor.yaml
│
└── .gitlab-ci.yml
```

Pipeline:

```text
Git Push
   ↓
GitLab CI
   ↓
Build
   ↓
Test
   ↓
Docker Build
   ↓
Push Image
   ↓
Deploy to Kubernetes
   ↓
Application Running
   ↓
Prometheus Scrapes Metrics
   ↓
Grafana Dashboard
   ↓
Alert if Problem
```

Then we'll deliberately create failures:

```text
CPU spike
Pod crash
Pod unavailable
Application error
High latency
Deployment failure
```

and observe how:

```text
Kubernetes
    ↓
Prometheus
    ↓
Alertmanager
    ↓
Grafana
```

responds.

---

# 🟢 Recommended Lesson Sequence

Since you prefer learning **step-by-step rather than getting everything at once**, I'd structure our lessons like this:

### Prometheus

**Lesson 1:** Monitoring & Observability Fundamentals
**Lesson 2:** Prometheus Architecture
**Lesson 3:** Install Prometheus with Docker
**Lesson 4:** Prometheus Configuration
**Lesson 5:** Targets and Scraping
**Lesson 6:** Exporters
**Lesson 7:** Metrics Types
**Lesson 8:** Labels and Time Series
**Lesson 9:** PromQL Basics
**Lesson 10:** PromQL Advanced
**Lesson 11:** Recording Rules

### Grafana

**Lesson 12:** Install Grafana
**Lesson 13:** Connect Grafana to Prometheus
**Lesson 14:** Panels and Visualizations
**Lesson 15:** Dashboards
**Lesson 16:** Variables and Filters
**Lesson 17:** Build a complete dashboard

### Kubernetes

**Lesson 18:** Kubernetes Monitoring Architecture
**Lesson 19:** kube-state-metrics
**Lesson 20:** Node Exporter
**Lesson 21:** cAdvisor / Container Metrics
**Lesson 22:** Prometheus Operator
**Lesson 23:** ServiceMonitor
**Lesson 24:** PodMonitor
**Lesson 25:** Kubernetes Dashboard

### Alerting

**Lesson 26:** Prometheus Alert Rules
**Lesson 27:** Alert States
**Lesson 28:** Alertmanager
**Lesson 29:** Routing & Grouping
**Lesson 30:** Silencing & Inhibition
**Lesson 31:** Email/Teams/Slack Notifications
**Lesson 32:** Grafana Alerting

### Application Monitoring

**Lesson 33:** Application `/metrics`
**Lesson 34:** HTTP Metrics
**Lesson 35:** Application Errors
**Lesson 36:** Latency Monitoring
**Lesson 37:** SLI/SLO Basics

### GitLab

**Lesson 38:** GitLab CI/CD + Kubernetes
**Lesson 39:** Build & Push Docker Image
**Lesson 40:** Deploy to Kubernetes
**Lesson 41:** Prometheus monitoring of deployment
**Lesson 42:** CI/CD + Monitoring + Alerting

### Azure — Final Stage

**Lesson 43:** AKS Monitoring Architecture
**Lesson 44:** Azure Managed Prometheus
**Lesson 45:** Azure Managed Grafana
**Lesson 46:** AKS + Prometheus + Grafana
**Lesson 47:** GitLab → AKS → Monitoring
**Lesson 48:** Production Monitoring Project

## 🎯 What you should be able to do at the end

You should be able to look at this architecture and explain **every component**:

```text
                         GitLab
                           │
                      GitLab CI/CD
                           │
                           ▼
                    ┌──────────────┐
                    │ Kubernetes   │
                    │    Cluster   │
                    └──────┬───────┘
                           │
             ┌─────────────┼─────────────┐
             │             │             │
           Nodes          Pods       Applications
             │             │             │
             └─────────────┼─────────────┘
                           │
                    Metrics / Exporters
                           │
                           ▼
                    ┌──────────────┐
                    │  Prometheus  │
                    └──────┬───────┘
                           │
                     PromQL / Rules
                           │
              ┌────────────┴────────────┐
              │                         │
           Grafana                 Alertmanager
              │                         │
         Dashboards              Notifications
              │                         │
              ▼                         ▼
        Visualization          Email/Teams/Slack
```

**I suggest we start with Lesson 1: Monitoring & Observability Fundamentals**, and then immediately move into a hands-on Prometheus installation. I can give you **only Lesson 1 first**, including a small Kubernetes example and exercises, and we can proceed lesson-by-lesson.
