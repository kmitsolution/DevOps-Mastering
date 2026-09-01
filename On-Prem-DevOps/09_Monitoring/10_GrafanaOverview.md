# Lesson 10 — Grafana Fundamentals

Now we move from **Prometheus querying → visualization**.

You've already learned how Prometheus collects and stores metrics and how PromQL retrieves them.

Today we'll connect that knowledge to Grafana.

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
    ↓
Dashboard
```

---

# 1. What is Grafana?

Grafana is a **visualization and observability platform**.

Prometheus gives you data like:

```text
namespace="monitoring" pod="prometheus-0" → CPU = 1.2
namespace="production" pod="api-1"       → CPU = 0.4
```

Grafana turns that data into:

```text
┌──────────────────────────────────────┐
│       Kubernetes Dashboard           │
├──────────────────────────────────────┤
│                                      │
│ Running Pods       42                │
│                                      │
│ CPU Usage          62%               │
│ ███████████████░░░░░                 │
│                                      │
│ Memory             5.4 GB            │
│                                      │
│ Pod Restarts       3                 │
└──────────────────────────────────────┘
```

So remember:

```text
Prometheus → data
Grafana    → visualization
```

---

# 2. Our Grafana Architecture

Because we installed `kube-prometheus-stack`, Grafana is already running inside Kubernetes.

Our architecture is:

```text
              Kubernetes Cluster
                     │
                     ▼
               Prometheus
                     │
                  Metrics
                     │
                  PromQL
                     │
                     ▼
                 Grafana
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
        Panel      Panel      Panel
          │          │          │
          ▼          ▼          ▼
         CPU       Memory      Pods
```

---

# 3. Access Grafana

If Grafana isn't currently exposed, run:

```bash
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
```

Then open:

```text
http://localhost:3000
```

Login:

```text
Username:
admin
```

For the password:

```bash
kubectl get secret -n monitoring monitoring-grafana \
  -o jsonpath="{.data.admin-password}" | base64 --decode
```

---

# 4. Understand the Grafana UI

After login, you'll see the Grafana interface.

The important areas for us are:

```text
Grafana
│
├── Dashboards
│
├── Explore
│
├── Alerting
│
└── Connections
```

We'll primarily work with:

```text
Explore
Dashboards
Alerting
```

---

# 5. What Is a Data Source?

Grafana itself doesn't normally know your Kubernetes metrics.

It needs to know:

> Where is my data?

That's the purpose of a **Data Source**.

Our data source is:

```text
Prometheus
```

Architecture:

```text
Grafana
   │
   │ Data Source
   ▼
Prometheus
   │
   ▼
Metrics
```

---

# 6. Check the Prometheus Data Source

Go to:

```text
Connections
   ↓
Data Sources
```

You should find:

```text
Prometheus
```

Because `kube-prometheus-stack` normally configures Grafana with Prometheus automatically.

Open it.

You'll see a URL similar to:

```text
http://monitoring-kube-prometheus-prometheus:9090
```

The exact name may differ in your installation.

---

# 7. Why Isn't It `localhost:9090`?

This is an important Kubernetes concept.

From **your computer**, you access:

```text
localhost:9090
```

because you created a port-forward.

But Grafana is running **inside Kubernetes**.

Grafana should communicate with Prometheus using the Kubernetes Service:

```text
Grafana
   │
   │ Kubernetes DNS
   ▼
Prometheus Service
   │
   ▼
Prometheus Pod
```

So the Grafana datasource might look like:

```text
http://<prometheus-service>:9090
```

not:

```text
http://localhost:9090
```

Because inside the Grafana container:

```text
localhost
```

means:

> the Grafana container itself.

Not your laptop.

---

# 8. Test the Data Source

In the Prometheus data source configuration, use:

```text
Save & test
```

You should get a successful connection.

Conceptually:

```text
Grafana
   │
   │ HTTP
   ▼
Prometheus
   │
   ▼
Success
```

---

# 9. Grafana Explore

Now go to:

```text
Explore
```

Select:

```text
Prometheus
```

as the data source.

You should get a query editor.

This is where the PromQL knowledge from Lessons 7–9 becomes useful.

---

# 10. Your First Grafana Query

Enter:

```promql
up
```

Run the query.

You should see the Prometheus data.

Instead of seeing it only as raw Prometheus output, Grafana can visualize it.

You can switch visualization types.

---

# 11. Stat Panel

A **Stat** panel is useful when you want one important number.

Example:

```text
┌───────────────────────┐
│   Running Targets     │
│                       │
│          15           │
│                       │
└───────────────────────┘
```

For example:

```promql
count(up == 1)
```

This could become:

```text
Running Targets
      15
```

---

# 12. Gauge Panel

A gauge is useful for percentages.

For example:

```text
┌─────────────────────┐
│    CPU Usage        │
│                     │
│       72%           │
│      ◉◉◉◉◉         │
└─────────────────────┘
```

We'll use gauges for things like:

```text
CPU %
Memory %
Disk %
```

---

# 13. Time Series Panel

This is one of the most important Grafana visualizations.

Suppose CPU changes over time:

```text
CPU %
100│
 80│             ●
 60│       ●─────●
 40│ ●─────●
 20│
   └────────────────────
     10  20  30  40  50
```

A **Time series** panel lets you see trends.

Use it for:

```text
CPU over time
Memory over time
Requests/sec
Errors/sec
Latency
```

---

# 14. Table Panel

A table is useful when you want detailed information.

For example:

```text
┌────────────┬─────────────┬──────────┐
│ Namespace  │ Pod         │ CPU      │
├────────────┼─────────────┼──────────┤
│ production │ api-1       │ 0.42     │
│ production │ api-2       │ 0.35     │
│ monitoring │ grafana     │ 0.20     │
└────────────┴─────────────┴──────────┘
```

We'll use this for:

```text
Top Pods
Pod restarts
Node status
Deployment status
```

---

# 15. Build Your First Dashboard

Now let's create a dashboard.

Go to:

```text
Dashboards
   ↓
New
   ↓
New dashboard
```

Choose:

```text
Add visualization
```

Select:

```text
Prometheus
```

as the data source.

---

# 16. Panel 1 — Running Targets

Use:

```promql
count(up == 1)
```

Choose:

```text
Visualization → Stat
```

Title:

```text
Running Targets
```

You should get something like:

```text
┌─────────────────────────┐
│ Running Targets         │
│                         │
│           15            │
└─────────────────────────┘
```

Save the panel.

---

# 17. Panel 2 — Running Pods

Create another visualization.

Query:

```promql
count(
  kube_pod_status_phase{
    phase="Running"
  }
)
```

Visualization:

```text
Stat
```

Title:

```text
Running Pods
```

Now your dashboard might look like:

```text
┌───────────────────┐  ┌───────────────────┐
│ Running Targets   │  │ Running Pods      │
│                   │  │                   │
│       15          │  │       42          │
└───────────────────┘  └───────────────────┘
```

---

# 18. Panel 3 — CPU by Namespace

Create another panel.

Use:

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

Choose:

```text
Time series
```

Title:

```text
CPU Usage by Namespace
```

You'll see multiple lines:

```text
CPU
 │
 │        production
 │       /──────────
 │      /
 │ ───── monitoring
 │
 └────────────────────
        Time
```

---

# 19. Panel 4 — Memory by Namespace

Query:

```promql
sum by (namespace) (
  container_memory_working_set_bytes{
    namespace!="",
    pod!="",
    container!=""
  }
)
```

Choose:

```text
Time series
```

Title:

```text
Memory Usage by Namespace
```

Grafana will plot memory usage over time.

---

# 20. Panel 5 — Top 5 CPU Pods

Use:

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

Choose:

```text
Table
```

Title:

```text
Top 5 CPU Consuming Pods
```

This is already becoming a useful Kubernetes dashboard.

---

# 21. Panel 6 — Pod Restarts

Use:

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

Choose:

```text
Table
```

Title:

```text
Top 5 Pod Restarts
```

Now you have an operational dashboard.

---

# 22. Dashboard Structure

Your first dashboard should eventually look like:

```text
┌───────────────────────────────────────────────────────┐
│            Kubernetes Cluster Overview                │
├───────────────────────┬───────────────────────────────┤
│ Running Targets       │ Running Pods                  │
│       15              │       42                      │
├───────────────────────┴───────────────────────────────┤
│                                                       │
│             CPU Usage by Namespace                   │
│                                                       │
│       ╱╲       ╱──────╲                              │
│  ────╱──╲─────╱────────╲────                         │
│                                                       │
├───────────────────────────────────────────────────────┤
│                                                       │
│            Memory Usage by Namespace                 │
│                                                       │
├───────────────────────────────────────────────────────┤
│ Top CPU Pods                    │ Pod Restarts        │
│ ─────────────────────────       │ ───────────────     │
│ api-1          0.8              │ api-3      5        │
│ api-2          0.6              │ worker-2   3        │
│ worker-1       0.4              │ api-1      2        │
└───────────────────────────────────────────────────────┘
```

That's the beginning of a real monitoring dashboard.

---

# 23. Dashboard vs Explore

Understand this difference.

### Explore

Used for:

```text
Testing queries
Troubleshooting
Investigating metrics
Experimenting with PromQL
```

Think:

> **Explore = investigation**

### Dashboard

Used for:

```text
Permanent visualization
Operations
NOC
DevOps
Management
```

Think:

> **Dashboard = monitoring**

So your workflow should be:

```text
Explore
   ↓
Develop PromQL
   ↓
Validate result
   ↓
Create Panel
   ↓
Dashboard
```

---

# 24. Why PromQL Knowledge Matters

Grafana doesn't magically understand:

> Show me the top CPU-consuming Kubernetes Pods.

You provide the query:

```promql
topk(
  5,
  sum by (namespace, pod) (
    rate(
      container_cpu_usage_seconds_total[5m]
    )
  )
)
```

Grafana then visualizes the result.

So:

```text
PromQL
  ↓
Data
  ↓
Grafana
  ↓
Visualization
```

This is why the first nine lessons focused heavily on Prometheus.

---

# 25. Time Range

Grafana dashboards normally have a time selector.

For example:

```text
Last 5 minutes
Last 15 minutes
Last 1 hour
Last 6 hours
Last 24 hours
```

For example:

```text
CPU
 │
 │             ╭──╮
 │       ╭─────╯  ╰──
 │───────╯
 └─────────────────────
    Last 1 hour
```

When the time range changes, Grafana requests the corresponding data from Prometheus.

---

# 26. Refresh Interval

You can also configure dashboard refresh:

```text
5s
10s
30s
1m
5m
```

For Kubernetes monitoring, something like:

```text
30s
```

or:

```text
1m
```

is often reasonable for a learning dashboard.

Remember:

> Refreshing Grafana doesn't mean Prometheus suddenly collects metrics faster.

Prometheus has its own scrape interval.

For example:

```text
Prometheus scrape → every 15s

Grafana refresh → every 30s
```

These are two different things.

---

# 27. Very Important Architecture

You should now understand:

```text
                      Kubernetes
                          │
                          ▼
                      Metrics
                          │
              ┌───────────┼───────────┐
              │           │           │
             Node        K8s        App
           Exporter     State     /metrics
              │           │           │
              └───────────┼───────────┘
                          ▼
                     Prometheus
                          │
                         TSDB
                          │
                       PromQL
                          │
                          ▼
                       Grafana
                          │
             ┌────────────┼─────────────┐
             ▼            ▼             ▼
           Stat       Time Series     Table
             │            │             │
             └────────────┼─────────────┘
                          ▼
                     Dashboard
```

---

# 🧪 Lesson 10 Lab

Let's keep this hands-on.

### Step 1

Access Grafana:

```text
http://localhost:3000
```

### Step 2

Go to:

```text
Connections → Data Sources
```

Verify:

```text
Prometheus
```

### Step 3

Go to:

```text
Explore
```

Select Prometheus.

Run:

```promql
up
```

### Step 4

Run:

```promql
count(up == 1)
```

### Step 5

Run:

```promql
count(
  kube_pod_status_phase{
    phase="Running"
  }
)
```

### Step 6

Create a dashboard with at least these two panels:

```text
Panel 1 → Running Targets
Panel 2 → Running Pods
```

Use **Stat** visualization for both.

---

# 🎯 Lesson 10 Goal

Don't try to create the entire dashboard today.

Your goal is simply:

```text
Prometheus
     ↓
PromQL
     ↓
Grafana Explore
     ↓
Panel
     ↓
Dashboard
```

Once that flow is comfortable, we'll build the dashboard progressively.

---

# Next — Lesson 11: Grafana Kubernetes Dashboard

In the next lesson we'll build a proper dashboard with:

```text
┌─────────────────────────────────────────┐
│         Kubernetes Cluster              │
├────────────┬────────────┬───────────────┤
│ Nodes      │ Pods       │ Namespaces    │
├────────────┴────────────┴───────────────┤
│ CPU Usage                                │
├─────────────────────────────────────────┤
│ Memory Usage                             │
├─────────────────────────────────────────┤
│ Top CPU Pods                             │
├─────────────────────────────────────────┤
│ Top Memory Pods                          │
├─────────────────────────────────────────┤
│ Pod Restarts                             │
└─────────────────────────────────────────┘
```

We'll also introduce **Grafana variables**, so instead of creating separate dashboards for every namespace, you'll be able to select:

```text
Namespace: [ production ▼ ]
```

and the entire dashboard dynamically changes to that namespace.
