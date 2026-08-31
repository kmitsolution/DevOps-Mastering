# Lesson 4 — Understand the kube-prometheus-stack Components

Great. At this point, assume your `kube-prometheus-stack` is installed and running.

Before we start writing PromQL or building Grafana dashboards, you need to understand **what Helm actually installed**.

The most important architecture is:

```text
                         Kubernetes Cluster
                                │
                    ┌───────────┴───────────┐
                    │                       │
              Prometheus Operator      kube-state-metrics
                    │                       │
                    │                       │
                    ▼                       ▼
               Prometheus              K8s Object
                    │                    Metrics
                    │
        ┌───────────┼────────────┐
        │           │            │
        ▼           ▼            ▼
   Node Exporter   Pods       Applications
        │
        ▼
   Node Metrics
                    │
                    ▼
               Alertmanager
                    │
                    ▼
            Email / Slack / Teams

                    +
                    
                 Grafana
                    │
                    ▼
              Visualization
```

Let's understand each component.

---

# 1. Prometheus

This is the **main metrics engine**.

It:

* Scrapes metrics
* Stores time-series data
* Executes PromQL
* Evaluates recording rules
* Evaluates alert rules

Conceptually:

```text
Targets
   │
   │ /metrics
   ▼
Prometheus
   │
   ├── Store metrics
   ├── Query metrics
   └── Evaluate rules
```

Check it:

```bash
kubectl get pods -n monitoring
```

You'll find a pod with something similar to:

```text
prometheus-monitoring-kube-prometheus-prometheus-0
```

The exact name may differ.

---

# 2. Grafana

Grafana is our **visualization layer**.

```text
Prometheus
     │
     │ PromQL
     ▼
  Grafana
     │
     ▼
Dashboards
```

For example:

```text
┌───────────────────────────────────┐
│ Kubernetes Cluster                │
├───────────────────────────────────┤
│                                   │
│ CPU Usage       ███████░░  72%    │
│ Memory Usage    █████░░░░  51%    │
│                                   │
│ Pods            32                │
│ Nodes           3                 │
│                                   │
└───────────────────────────────────┘
```

---

# 3. Prometheus Operator

This is one of the **most important concepts for Kubernetes**.

Normally, you could configure Prometheus using:

```text
prometheus.yml
```

But Kubernetes is dynamic.

Instead, the Prometheus Operator allows you to define monitoring configuration using **Kubernetes resources**.

Think:

```text
Traditional Prometheus

prometheus.yml
       ↓
Prometheus
```

versus:

```text
Kubernetes

Kubernetes YAML
       ↓
Prometheus Operator
       ↓
Prometheus
```

The Operator essentially says:

> "You describe what you want to monitor using Kubernetes resources, and I'll configure Prometheus."

---

# 4. Custom Resources

The Operator introduces Kubernetes CRDs.

You will eventually work with:

```text
Prometheus
Alertmanager
ServiceMonitor
PodMonitor
PrometheusRule
Probe
```

You can check them:

```bash
kubectl get crd | grep monitoring
```

You'll see resources similar to:

```text
alertmanagerconfigs.monitoring.coreos.com
alertmanagers.monitoring.coreos.com
podmonitors.monitoring.coreos.com
prometheuses.monitoring.coreos.com
prometheusrules.monitoring.coreos.com
servicemonitors.monitoring.coreos.com
```

Don't worry about all of them yet.

The three you'll use heavily are:

```text
ServiceMonitor
PodMonitor
PrometheusRule
```

---

# 5. ServiceMonitor

Suppose you have:

```text
my-api
   │
   ▼
Service
   │
   ▼
Pod
```

You want Prometheus to monitor it.

You can create a:

```yaml
kind: ServiceMonitor
```

Conceptually:

```text
ServiceMonitor
      │
      │ tells Operator
      ▼
Prometheus
      │
      │ scrape
      ▼
Service
      │
      ▼
Application /metrics
```

This is much better than manually adding every Pod IP.

---

# 6. PodMonitor

PodMonitor is similar, but targets Pods directly.

```text
PodMonitor
    │
    ▼
Prometheus
    │
    ▼
Pods
```

You'll learn when to use `ServiceMonitor` versus `PodMonitor` later.

For now:

```text
ServiceMonitor → monitor through Services

PodMonitor → monitor Pods directly
```

---

# 7. kube-state-metrics

This component is particularly important for Kubernetes.

It converts Kubernetes object state into Prometheus metrics.

For example, Kubernetes knows:

```text
Deployment:
Desired replicas = 3
Available replicas = 2
```

kube-state-metrics exposes that information as metrics.

Architecture:

```text
Kubernetes API
       │
       ▼
kube-state-metrics
       │
       ▼
    /metrics
       │
       ▼
   Prometheus
```

Example metrics:

```text
kube_pod_info
kube_pod_status_phase
kube_deployment_spec_replicas
kube_deployment_status_replicas_available
```

This allows Prometheus to understand **Kubernetes state**.

---

# 8. Node Exporter

Node Exporter monitors the underlying machines.

```text
Worker Node
    │
    ├── CPU
    ├── Memory
    ├── Disk
    ├── Filesystem
    └── Network
          │
          ▼
    Node Exporter
          │
          ▼
      Prometheus
```

You'll normally see one Node Exporter Pod per Kubernetes node.

Check:

```bash
kubectl get pods -n monitoring -l app.kubernetes.io/name=node-exporter
```

Because it's generally deployed as a **DaemonSet**, each node gets an instance.

---

# 9. Why One Node Exporter Per Node?

Suppose you have:

```text
3 Worker Nodes

Node 1
Node 2
Node 3
```

Node Exporter:

```text
Node 1 → Node Exporter
Node 2 → Node Exporter
Node 3 → Node Exporter
```

Why?

Because each exporter needs to report the metrics of its own machine.

So:

```text
DaemonSet
   ↓
One Pod per Node
```

This is a useful Kubernetes concept as well.

---

# 10. Alertmanager

Prometheus can detect an alert condition.

For example:

```text
CPU > 80%
```

But Alertmanager handles the notification workflow.

```text
Prometheus
    │
    │ Alert
    ▼
Alertmanager
    │
    ├── Group alerts
    ├── Route alerts
    ├── Silence alerts
    └── Send notifications
```

Later:

```text
Alertmanager
     │
     ├── Email
     ├── Slack
     ├── Microsoft Teams
     └── Other receivers
```

---

# 11. The Difference Between Prometheus and Alertmanager

This is a common interview question.

### Prometheus

Answers:

> Is there a problem?

Example:

```text
CPU > 80%
```

### Alertmanager

Answers:

> Who should be notified, and how?

Example:

```text
severity=critical
        ↓
DevOps Team
        ↓
Microsoft Teams
```

So:

```text
Prometheus = Detect

Alertmanager = Manage and notify
```

---

# 12. Let's Look at the Pods

Run:

```bash
kubectl get pods -n monitoring
```

You may see something like:

```text
NAME                                             READY
alertmanager-monitoring-...-0                    2/2
monitoring-grafana-...                           3/3
monitoring-kube-prometheus-operator-...         1/1
monitoring-kube-state-metrics-...               1/1
monitoring-prometheus-node-exporter-...         1/1
prometheus-monitoring-...-0                      2/2
```

Don't worry if the names aren't exactly the same.

Let's map them.

| Component           | Purpose                          |
| ------------------- | -------------------------------- |
| Prometheus          | Collect/store/query metrics      |
| Grafana             | Visualization                    |
| Prometheus Operator | Manages Prometheus in Kubernetes |
| kube-state-metrics  | Kubernetes object/state metrics  |
| Node Exporter       | Node/OS metrics                  |
| Alertmanager        | Alert routing and notifications  |

---

# 13. Understand the Data Flow

This is the most important part of today's lesson.

Suppose we want to monitor:

```text
Worker Node
```

The flow is:

```text
Worker Node
     │
     ▼
Node Exporter
     │
     │ /metrics
     ▼
Prometheus
     │
     │ stores metrics
     ▼
Prometheus TSDB
     │
     │ PromQL
     ▼
Grafana
     │
     ▼
Dashboard
```

Now suppose we want to monitor:

```text
Deployment replicas
```

The flow is:

```text
Kubernetes API
     │
     ▼
kube-state-metrics
     │
     │ /metrics
     ▼
Prometheus
     │
     ▼
Grafana
```

Notice that these are **different sources of metrics**.

---

# 14. Three Different Types of Information

This distinction will help you enormously.

### Infrastructure metrics

From:

```text
Node Exporter
```

Examples:

```text
CPU
Memory
Disk
Network
```

### Kubernetes state metrics

From:

```text
kube-state-metrics
```

Examples:

```text
Pod status
Deployment replicas
DaemonSet status
Job status
```

### Application metrics

From:

```text
Application /metrics
```

Examples:

```text
HTTP requests
HTTP errors
Latency
Business metrics
```

So:

```text
                    Prometheus
                        ▲
             ┌──────────┼──────────┐
             │          │          │
             │          │          │
       Node Exporter   KSM     Application
             │          │          │
             ▼          ▼          ▼
         Infrastructure K8s     App Metrics
```

---

# 15. Your First Important Kubernetes Monitoring Question

Suppose you run:

```bash
kubectl get pods
```

and see:

```text
my-api-abc123     1/1   Running
my-api-def456     1/1   Running
my-api-xyz789     0/1   CrashLoopBackOff
```

Which component helps Prometheus obtain Kubernetes object state such as Pod status?

**Answer: kube-state-metrics.**

But if you ask:

> How much CPU is the underlying node consuming?

That's infrastructure/node metrics, typically obtained through **Node Exporter**.

---

# 16. One More Important Concept: cAdvisor

You'll also encounter **cAdvisor/container metrics**.

Think:

```text
Node Exporter
     ↓
Node-level metrics

cAdvisor
     ↓
Container-level resource metrics
```

For example:

```text
Container CPU
Container Memory
Container filesystem
Container network
```

So eventually your monitoring picture becomes:

```text
                  Prometheus
                      ▲
        ┌─────────────┼─────────────┐
        │             │             │
        │             │             │
 Node Exporter   kube-state-      cAdvisor /
                 metrics          container metrics
        │             │             │
        ▼             ▼             ▼
      Node          K8s State     Containers
```

---

# 17. Practical Exploration

Now let's inspect your cluster.

### Command 1 — Pods

```bash
kubectl get pods -n monitoring
```

### Command 2 — Services

```bash
kubectl get svc -n monitoring
```

### Command 3 — CRDs

```bash
kubectl get crd | grep monitoring.coreos.com
```

### Command 4 — ServiceMonitors

```bash
kubectl get servicemonitor -A
```

### Command 5 — PodMonitors

```bash
kubectl get podmonitor -A
```

### Command 6 — Prometheus

```bash
kubectl get prometheus -n monitoring
```

### Command 7 — Alertmanager

```bash
kubectl get alertmanager -n monitoring
```

These commands are more important than memorizing YAML at this stage.

---

# 🧠 Lesson 4 Mental Model

You should now be able to explain:

```text
                  Kubernetes
                      │
       ┌──────────────┼──────────────┐
       │              │              │
       ▼              ▼              ▼
      Nodes          K8s          Applications
       │             Objects           │
       ▼              │                ▼
Node Exporter         ▼             /metrics
       │        kube-state-            │
       │          metrics              │
       └──────────────┬─────────────────┘
                      ▼
                 Prometheus
                      │
               ┌──────┴──────┐
               │             │
            PromQL        Alert Rules
               │             │
               ▼             ▼
            Grafana      Alertmanager
               │             │
               ▼             ▼
           Dashboard    Notifications
```

---

## 🎯 Your Exercise

Run these **six commands**:

```bash
kubectl get pods -n monitoring
```

```bash
kubectl get svc -n monitoring
```

```bash
kubectl get servicemonitor -A
```

```bash
kubectl get podmonitor -A
```

```bash
kubectl get prometheus -n monitoring
```

```bash
kubectl get alertmanager -n monitoring
```

Then the next lesson will be:

# **Lesson 5 — Prometheus Targets & Scraping**

We'll open the Prometheus UI and understand:

```text
Status
  ↓
Targets
  ↓
UP / DOWN
  ↓
Endpoint
  ↓
/metrics
  ↓
Scrape interval
  ↓
Labels
```

**This is where we'll make the architecture from Lessons 1–4 visible in your actual Kubernetes cluster.**
