# Lesson 11 — Build a Kubernetes Dashboard in Grafana

Great. Now that you understand **Prometheus + PromQL + Grafana basics**, let's build a proper Kubernetes monitoring dashboard.

We'll do it **step-by-step**, and I'll explicitly tell you **where to navigate in Grafana** at every stage.

Our dashboard will eventually look conceptually like:

```text
┌─────────────────────────────────────────────────────────┐
│              Kubernetes Cluster Overview                │
├──────────────┬──────────────┬──────────────┬─────────────┤
│ Running Pods │ Nodes        │ Targets      │ CPU %       │
│     42       │      3       │     15       │    65%      │
├──────────────┴──────────────┴──────────────┴─────────────┤
│                                                         │
│              CPU Usage by Namespace                    │
│                                                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│             Memory Usage by Namespace                  │
│                                                         │
├────────────────────────────┬────────────────────────────┤
│ Top CPU Pods               │ Pod Restarts               │
│                            │                            │
│ api-1       0.82           │ api-3          5           │
│ api-2       0.63           │ worker-2       3           │
│                            │                            │
└────────────────────────────┴────────────────────────────┘
```

But **don't create everything at once**.

Today we'll create the first few panels and understand how Grafana dashboards work.

---

# Part 1 — Open Grafana

Make sure Grafana is running.

If you haven't already:

```bash
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
```

Open:

```text
http://localhost:3000
```

Login with:

```text
Username: admin
Password: <your Grafana password>
```

---

# Part 2 — Create a New Dashboard

In Grafana, navigate:

```text
Left Sidebar
   ↓
Dashboards
   ↓
New
   ↓
New dashboard
```

You should now see an empty dashboard.

Look for:

```text
Add visualization
```

Click it.

---

# Part 3 — Select Prometheus

Grafana will ask you to select a data source.

Select:

```text
Prometheus
```

You should now be inside the **panel editor**.

You'll generally see sections such as:

```text
Query
Transform
Visualization
```

The exact position can vary slightly with Grafana versions.

---

# Part 4 — First Panel: Running Targets

Let's create our first useful panel.

## Navigate

```text
Dashboards
 → New dashboard
 → Add visualization
```

Select:

```text
Prometheus
```

In the query editor, enter:

```promql
count(up == 1)
```

Click:

```text
Run queries
```

or the equivalent **Run query** button in your Grafana version.

You should get a number such as:

```text
15
```

---

# Part 5 — Change Visualization to Stat

Now look for the **Visualization** selector in the panel editor.

Navigate:

```text
Panel Editor
   ↓
Visualization
```

Select:

```text
Stat
```

If you don't immediately see it, use the visualization search box and type:

```text
Stat
```

You should see the Stat visualization.

The result should look approximately like:

```text
┌──────────────────────┐
│        15            │
│                      │
│ Running Targets      │
└──────────────────────┘
```

---

# Part 6 — Give the Panel a Title

In the panel editor, find the panel options/title area.

Depending on your Grafana version, you may find it under:

```text
Panel options
```

Set the title to:

```text
Running Targets
```

Then click:

```text
Save
```

or:

```text
Apply
```

depending on the version.

---

# Part 7 — Save the Dashboard

At the top-right of the dashboard, click:

```text
Save dashboard
```

Give it a name:

```text
Kubernetes Cluster Overview
```

Then save it.

Now you have your first Grafana dashboard.

---

# Part 8 — Add Running Pods

Now we'll add another Stat panel.

Navigate:

```text
Dashboard
   ↓
Add
   ↓
Visualization
```

Again select:

```text
Prometheus
```

Use:

```promql
count(
  kube_pod_status_phase{
    phase="Running"
  }
)
```

Click:

```text
Run queries
```

---

# Part 9 — Select Stat

Navigate:

```text
Visualization
   ↓
Stat
```

Set the title:

```text
Running Pods
```

Click:

```text
Apply
```

Now your dashboard should have:

```text
┌─────────────────────┐   ┌─────────────────────┐
│ Running Targets     │   │ Running Pods        │
│                     │   │                     │
│        15           │   │        42           │
└─────────────────────┘   └─────────────────────┘
```

---

# Part 10 — Add Node Count

Let's add another Stat.

Navigate:

```text
Dashboard
   ↓
Add
   ↓
Visualization
```

Use:

```promql
count(
  kube_node_info
)
```

Visualization:

```text
Stat
```

Title:

```text
Kubernetes Nodes
```

Click:

```text
Apply
```

You should now have three panels:

```text
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│   Targets    │ │     Pods     │ │    Nodes     │
│      15      │ │      42      │ │       3      │
└──────────────┘ └──────────────┘ └──────────────┘
```

---

# Part 11 — Add CPU Usage by Namespace

Now we move from **Stat → Time Series**.

Navigate:

```text
Dashboard
   ↓
Add
   ↓
Visualization
```

Select:

```text
Prometheus
```

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

Click:

```text
Run queries
```

---

# Part 12 — Select Time Series

Navigate:

```text
Visualization
   ↓
Time series
```

If you don't immediately see it, search:

```text
Time series
```

You should get a graph similar to:

```text
CPU
 │
 │             production
 │          ╭────────────
 │       ╭──╯
 │  ╭────╯
 │──╯
 └────────────────────────
             Time
```

Set title:

```text
CPU Usage by Namespace
```

Then:

```text
Apply
```

---

# Part 13 — Why Time Series?

Remember the difference:

### Stat

Good for:

```text
Running Pods = 42
```

You care about the current number.

### Time Series

Good for:

```text
CPU
Memory
Requests
Latency
```

because you want to see how the value changes over time.

So:

```text
Stat
 ↓
Current value

Time Series
 ↓
Value over time
```

---

# Part 14 — Add Memory Usage by Namespace

Create another panel:

```text
Dashboard
   ↓
Add
   ↓
Visualization
```

Select:

```text
Prometheus
```

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

Select:

```text
Time series
```

Title:

```text
Memory Usage by Namespace
```

Click:

```text
Apply
```

---

# Part 15 — Change Memory Unit

The memory query returns bytes.

For example:

```text
4294967296
```

That's not very readable.

We want:

```text
4 GiB
```

In the panel editor, navigate to:

```text
Panel
   ↓
Field
   ↓
Standard options
   ↓
Unit
```

Select a bytes/information unit such as:

```text
Bytes → IEC
```

or the equivalent **bytes (IEC)** option in your Grafana version.

Now Grafana can display:

```text
4294967296
```

as approximately:

```text
4 GiB
```

This is an important Grafana skill:

> **Prometheus provides the number; Grafana controls how that number is displayed.**

---

# Part 16 — Add Top 5 CPU Pods

Create another panel:

```text
Dashboard
   ↓
Add
   ↓
Visualization
```

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

Now navigate:

```text
Visualization
   ↓
Table
```

Set title:

```text
Top 5 CPU Consuming Pods
```

Click:

```text
Apply
```

You'll get something similar to:

```text
┌────────────┬──────────────┬───────┐
│ namespace  │ pod          │ value │
├────────────┼──────────────┼───────┤
│ monitoring │ prometheus   │ 1.20  │
│ production │ api-1        │ 0.82  │
│ production │ api-2        │ 0.63  │
│ monitoring │ grafana      │ 0.40  │
│ production │ worker-1     │ 0.31  │
└────────────┴──────────────┴───────┘
```

---

# Part 17 — Add Top 5 Restarting Pods

Create another panel:

```text
Dashboard
   ↓
Add
   ↓
Visualization
```

Query:

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
Visualization
   ↓
Table
```

Title:

```text
Top 5 Pod Restarts — Last 1 Hour
```

Click:

```text
Apply
```

---

# Part 18 — Your Dashboard Now

You should now have something approximately like:

```text
┌───────────────────────────────────────────────────────┐
│          Kubernetes Cluster Overview                  │
├──────────────┬──────────────┬─────────────────────────┤
│ Running      │ Running Pods │ Kubernetes Nodes        │
│ Targets      │              │                         │
│    15        │     42       │        3                │
├──────────────┴──────────────┴─────────────────────────┤
│                                                       │
│             CPU Usage by Namespace                   │
│                                                       │
│                ╱──────╲                              │
│        ╭──────╯        ╲                             │
│   ─────╯                ╲────                        │
│                                                       │
├───────────────────────────────────────────────────────┤
│                                                       │
│            Memory Usage by Namespace                 │
│                                                       │
├────────────────────────────┬──────────────────────────┤
│ Top 5 CPU Pods             │ Top 5 Pod Restarts      │
│                            │                          │
│ api-1       0.82           │ api-3       5            │
│ api-2       0.63           │ worker-2    3            │
│ worker-1    0.31           │ api-1       2            │
└────────────────────────────┴──────────────────────────┘
```

---

# Part 19 — Arrange the Panels

Grafana lets you resize panels.

You can drag the edges/corners of a panel.

I recommend:

```text
Row 1:
┌────────┐ ┌────────┐ ┌────────┐
│ Targets│ │  Pods  │ │ Nodes  │
└────────┘ └────────┘ └────────┘

Row 2:
┌────────────────────────────────┐
│       CPU by Namespace         │
└────────────────────────────────┘

Row 3:
┌────────────────────────────────┐
│      Memory by Namespace       │
└────────────────────────────────┘

Row 4:
┌──────────────────┐ ┌───────────┐
│ Top CPU Pods     │ │ Restarts  │
└──────────────────┘ └───────────┘
```

This is a good basic monitoring dashboard layout.

---

# Part 20 — Set Dashboard Refresh

At the top-right of the dashboard, find the **refresh/time controls**.

You'll see something similar to:

```text
Last 15 minutes
```

and a refresh option.

For learning, select something like:

```text
30s
```

Now Grafana periodically refreshes the dashboard.

Remember:

```text
Prometheus scrape interval ≠ Grafana refresh interval
```

For example:

```text
Prometheus
   ↓
scrapes every 15 seconds

Grafana
   ↓
refreshes every 30 seconds
```

---

# Part 21 — Important: Save Again

After arranging the panels:

Navigate:

```text
Top-right
   ↓
Save dashboard
```

Save your changes.

---

# 🎯 Today's Practical Assignment

I don't want you to create anything else yet.

Your dashboard should contain these **six panels**:

| Panel                     | Visualization |
| ------------------------- | ------------- |
| Running Targets           | Stat          |
| Running Pods              | Stat          |
| Kubernetes Nodes          | Stat          |
| CPU Usage by Namespace    | Time series   |
| Memory Usage by Namespace | Time series   |
| Top 5 CPU Pods            | Table         |
| Top 5 Pod Restarts        | Table         |

Technically that's **7 panels**, which is exactly what I recommend for this lesson.

---

# 🧠 What You Learned

The workflow is now:

```text
Prometheus
     │
     │ PromQL
     ▼
Grafana
     │
     ▼
Panel
     │
     ├── Stat
     ├── Gauge
     ├── Time series
     └── Table
          │
          ▼
      Dashboard
```

And more importantly:

```text
Question
   ↓
Find Prometheus metric
   ↓
Write PromQL
   ↓
Test in Explore
   ↓
Create Grafana panel
   ↓
Choose visualization
   ↓
Configure units
   ↓
Add to dashboard
```

**This is the workflow I want you to practice.**

---

# Next Lesson — Lesson 12: Grafana Variables

This is where your dashboard becomes much more powerful.

Instead of making:

```text
Production Dashboard
Development Dashboard
Monitoring Dashboard
```

we'll make **one dashboard** with:

```text
Namespace: [ production ▼ ]
```

When you select:

```text
production
```

the panels show production.

When you select:

```text
monitoring
```

the same dashboard changes to monitoring.

We'll learn:

```text
Dashboard
   ↓
Variables
   ↓
Namespace dropdown
   ↓
PromQL filtering
   ↓
Dynamic dashboard
```

This is a very important skill for real-world Grafana usage.
