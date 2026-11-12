# Lesson 5 — Prometheus Targets & Scraping

Now we move from **architecture → actual monitoring**.

In Lesson 4, we learned:

```text
Node Exporter
kube-state-metrics
Application
      ↓
   /metrics
      ↓
 Prometheus
```

Today you'll see exactly **how Prometheus discovers and scrapes those targets**.

---

# 1. What Is a Prometheus Target?

A **target** is an endpoint that Prometheus should monitor.

For example:

```text
10.0.0.10:9100
```

might be a Node Exporter endpoint.

Prometheus accesses:

```text
http://10.0.0.10:9100/metrics
```

So:

```text
Target
   ↓
IP + Port
   ↓
/metrics
```

---

# 2. What Is Scraping?

**Scraping = Prometheus periodically fetching metrics from a target.**

Example:

```text
Prometheus
    │
    │ GET /metrics
    ▼
Node Exporter
    │
    │ Response
    ▼
Prometheus
```

Suppose your scrape interval is:

```text
15 seconds
```

Then:

```text
10:00:00 → scrape
10:00:15 → scrape
10:00:30 → scrape
10:00:45 → scrape
10:01:00 → scrape
```

Prometheus keeps collecting samples.

---

# 3. Let's Open Prometheus

If you haven't already exposed Prometheus, run:

```bash
kubectl port-forward -n monitoring \
  svc/monitoring-kube-prometheus-prometheus \
  9090:9090
```

You should see:

```text
Forwarding from 127.0.0.1:9090 -> 9090
```

Open:

```text
http://localhost:9090
```

You should see the Prometheus web UI.

---

# 4. Go to Status → Targets

In the Prometheus UI, go to:

```text
Status
   ↓
Targets
```

You should see several targets.

Depending on your installation, you may see jobs similar to:

```text
alertmanager
kube-apiserver
kube-controller-manager
kube-scheduler
kubelet
kube-state-metrics
node-exporter
prometheus
```

The exact list will depend on your cluster and chart version.

---

# 5. Understand UP and DOWN

You'll see something similar to:

```text
Endpoint                         State
──────────────────────────────────────
http://.../metrics               UP
http://.../metrics               UP
http://.../metrics               UP
```

### UP

Means:

> Prometheus successfully scraped the target.

### DOWN

Means:

> Prometheus attempted to scrape it but couldn't successfully collect metrics.

For example:

```text
Target                  State
node-exporter            UP
kube-state-metrics       UP
application              DOWN
```

This is one of the first places you'll investigate when monitoring breaks.

---

# 6. Target Details

Click on one of the targets.

You'll typically find information such as:

```text
Endpoint
State
Labels
Last Scrape
Scrape Duration
Error
```

For example:

```text
Endpoint:
http://10.244.x.x:9100/metrics

State:
UP

Last Scrape:
15 seconds ago

Scrape Duration:
20ms
```

---

# 7. Last Scrape

This is useful.

Suppose:

```text
Last Scrape: 5s ago
```

and:

```text
Scrape Interval: 15s
```

That means Prometheus is actively scraping the target.

If you see:

```text
Last Scrape: 10m ago
```

something is likely wrong.

---

# 8. Scrape Duration

You may also see:

```text
Scrape Duration: 25ms
```

This tells you approximately how long the scrape took.

Normally:

```text
10ms
20ms
30ms
```

isn't concerning by itself.

But if scraping takes a long time:

```text
5 seconds
10 seconds
```

that can become a monitoring problem.

---

# 9. How Does Prometheus Know About These Targets?

This is where Kubernetes becomes interesting.

With traditional Prometheus, you might configure:

```yaml
scrape_configs:
  - job_name: "node"
    static_configs:
      - targets:
          - "10.0.0.10:9100"
```

This is **static configuration**.

But Kubernetes doesn't work well with manually maintained IP addresses.

Pods can change:

```text
Pod A
10.244.1.5
```

then later:

```text
Pod A
10.244.2.8
```

So Kubernetes monitoring needs **service discovery**.

---

# 10. Kubernetes Service Discovery

Prometheus can discover Kubernetes objects.

Conceptually:

```text
                Kubernetes API
                      │
                      ▼
             Prometheus Discovery
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
        Nodes        Pods       Services
          │           │           │
          └───────────┼───────────┘
                      ▼
                  Targets
                      │
                      ▼
                  Prometheus
```

This means Prometheus can dynamically discover targets.

---

# 11. Prometheus Operator Makes This Easier

Since we're using:

```text
kube-prometheus-stack
```

we have the Prometheus Operator.

The Operator uses Kubernetes resources such as:

```text
ServiceMonitor
PodMonitor
```

to tell Prometheus what to scrape.

For example:

```text
Service
   │
   ▼
ServiceMonitor
   │
   ▼
Prometheus Operator
   │
   ▼
Prometheus configuration
   │
   ▼
Scraping
```

---

# 12. Let's Look at Your ServiceMonitors

Run:

```bash
kubectl get servicemonitor -A
```

You may see something similar to:

```text
NAMESPACE    NAME
monitoring   monitoring-kube-apiserver
monitoring   monitoring-kube-scheduler
monitoring   monitoring-kubelet
monitoring   monitoring-kube-prometheus-alertmanager
monitoring   monitoring-kube-prometheus-prometheus
monitoring   monitoring-kube-state-metrics
```

The names may differ.

These objects tell Prometheus:

> Here are Kubernetes Services that should be monitored.

---

# 13. Look Inside a ServiceMonitor

Pick one:

```bash
kubectl get servicemonitor -n monitoring
```

Then:

```bash
kubectl describe servicemonitor <NAME> -n monitoring
```

You'll see information about:

```text
Selector
Endpoints
Port
Path
Interval
Labels
```

One of the most important pieces is:

```text
Path: /metrics
```

That tells Prometheus where to obtain metrics.

---

# 14. The Complete ServiceMonitor Flow

Suppose we want to monitor kube-state-metrics.

The flow is:

```text
kube-state-metrics
        │
        │ exposes
        ▼
    /metrics
        ▲
        │
      Service
        ▲
        │
ServiceMonitor
        ▲
        │
Prometheus Operator
        │
        ▼
    Prometheus
```

This is an important architecture to understand.

---

# 15. ServiceMonitor Does NOT Collect Metrics

This is a common misunderstanding.

`ServiceMonitor` does not itself collect metrics.

It is basically a **declaration/configuration**.

Think:

```text
ServiceMonitor
     │
     │ "Monitor this service"
     ▼
Prometheus Operator
     │
     │ configures
     ▼
Prometheus
     │
     │ scrapes
     ▼
Target
```

---

# 16. What About PodMonitor?

PodMonitor works similarly.

```text
PodMonitor
    ↓
Prometheus Operator
    ↓
Prometheus
    ↓
Pod /metrics
```

The difference we'll explore later is whether you're discovering metrics through a Kubernetes Service or directly from Pods.

For now:

```text
ServiceMonitor → Service-based discovery

PodMonitor → Pod-based discovery
```

---

# 17. Let's Query Prometheus

Now go to:

```text
http://localhost:9090
```

Find the expression/query box.

Enter:

```promql
up
```

and click **Execute**.

You should get results.

Something like:

```text
up{job="prometheus", ...} 1

up{job="node-exporter", ...} 1

up{job="kube-state-metrics", ...} 1
```

---

# 18. What Does `up` Mean?

This is your **first important PromQL metric**.

```promql
up
```

returns:

```text
1 → target is up
0 → target is down
```

For example:

```text
up{job="prometheus"} 1
```

means:

> Prometheus successfully scraped the Prometheus target.

And:

```text
up{job="my-app"} 0
```

means:

> The target is currently failing to scrape.

---

# 19. Why Does `up` Have Labels?

You may see:

```text
up{
  job="node-exporter",
  instance="10.0.0.5:9100"
}
```

Don't worry about all the labels yet.

But understand this concept:

```text
Metric
   +
Labels
   ↓
Unique Time Series
```

For example:

```text
up{job="node-exporter",instance="10.0.0.5:9100"}
```

and:

```text
up{job="node-exporter",instance="10.0.0.6:9100"}
```

are different time series.

**Labels are going to be one of our next major lessons.**

---

# 20. Try These Queries

In Prometheus, try:

### Query 1

```promql
up
```

### Query 2

```promql
up == 1
```

This returns only healthy targets.

### Query 3

```promql
up == 0
```

This helps find targets that are down.

### Query 4

```promql
count(up)
```

This tells you how many `up` time series exist.

### Query 5

```promql
count(up == 1)
```

This counts currently healthy targets.

---

# 21. See the Metrics Directly

Prometheus targets expose metrics through HTTP.

For example, Node Exporter may expose:

```text
/metrics
```

You can actually inspect the endpoint.

Find your Node Exporter service:

```bash
kubectl get svc -n monitoring | grep node
```

You can also inspect the Pod:

```bash
kubectl get pods -n monitoring | grep node-exporter
```

Later we'll directly access `/metrics` and see the raw Prometheus metrics.

That exercise will make the whole architecture "click."

---

# 22. Important Architecture

At this point, your mental model should be:

```text
                   Kubernetes API
                         │
                         ▼
                Service Discovery
                         │
                         ▼
                Prometheus Operator
                         │
                  ServiceMonitor
                  /    |      \
                 /     |       \
                ▼      ▼        ▼
          kube-state  kubelet  other services
             metrics
                │
                │ /metrics
                ▼
            Prometheus
                │
                │ stores
                ▼
               TSDB
                │
                │ PromQL
                ▼
             Grafana
```

---

# 🧪 Lesson 5 Practical Exercise

Do these in order.

### 1. Check targets

Open:

```text
http://localhost:9090
```

Go to:

```text
Status → Targets
```

Identify:

* Prometheus target
* Node Exporter target
* kube-state-metrics target
* Kubernetes API target

---

### 2. Run

```promql
up
```

Observe the labels.

---

### 3. Run

```promql
up == 1
```

---

### 4. Run

```promql
up == 0
```

Ideally you should get no results.

---

### 5. Run

```promql
count(up)
```

---

### 6. Run

```promql
count(up == 1)
```

Compare the results.

---

### 7. Kubernetes

Run:

```bash
kubectl get servicemonitor -A
```

Then inspect one:

```bash
kubectl describe servicemonitor <NAME> -n <NAMESPACE>
```

Look specifically for:

```text
selector
endpoints
port
path
interval
```

---

# 🎯 What You Learned Today

You should now understand:

```text
Target
   ↓
Endpoint
   ↓
/metrics
   ↓
Scrape
   ↓
Prometheus
   ↓
TSDB
   ↓
PromQL
```

And in Kubernetes:

```text
Kubernetes
   ↓
Service Discovery
   ↓
ServiceMonitor / PodMonitor
   ↓
Prometheus Operator
   ↓
Prometheus
```

### Next: Lesson 6 — Understanding Metrics, Labels & Time Series

This is a **very important lesson** before we go deeper into PromQL.

We'll take an actual metric such as:

```promql
kube_pod_status_phase
```

and break it apart:

```text
Metric name
     ↓
Labels
     ↓
Label values
     ↓
Time series
     ↓
Samples
```

Then you'll understand **why the same metric name can represent hundreds or thousands of Kubernetes objects**.
