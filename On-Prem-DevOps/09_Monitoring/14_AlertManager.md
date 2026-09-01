# Lesson 14 — Alertmanager with Prometheus + Kubernetes

Now we move into one of the most important parts of monitoring:

```text
Metrics
   ↓
Prometheus
   ↓
Alert Rule
   ↓
Alert fires
   ↓
Alertmanager
   ↓
Notification
   ├── Email
   ├── Slack
   ├── Teams
   ├── Webhook
   └── Other receivers
```

Since you're using **`kube-prometheus-stack` with Helm**, you most likely already have **Alertmanager installed**. We will first verify it, then create a real Kubernetes alert.

---

# 1. What Does Alertmanager Actually Do?

A common misunderstanding is:

> Prometheus sends emails/Slack messages.

Normally, the architecture is:

```text
                 Prometheus
                     │
                     │ Alert fires
                     ▼
                Alertmanager
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
        Email       Slack     Webhook
```

### Prometheus

Prometheus answers:

> Is something wrong?

For example:

```text
CPU > 80%
```

### Alertmanager

Alertmanager answers:

> What should I do with this alert?

For example:

```text
Production CPU alert
        ↓
   Alertmanager
        ↓
      Slack
```

It also handles:

* Grouping alerts
* Routing alerts
* Silencing alerts
* Inhibition
* Deduplication
* Notification receivers

---

# 2. Check Alertmanager in Kubernetes

First navigate to your terminal.

Run:

```bash
kubectl get pods -n monitoring
```

You should see something similar to:

```text
alertmanager-monitoring-kube-prometheus-alertmanager-0
grafana-xxxxxxxx
prometheus-monitoring-kube-prometheus-prometheus-0
kube-state-metrics-xxxxxxxx
node-exporter-xxxxx
```

Look specifically for:

```text
alertmanager-...
```

---

# 3. Check Alertmanager Service

Run:

```bash
kubectl get svc -n monitoring
```

Look for something like:

```text
alertmanager-operated
```

or:

```text
monitoring-kube-prometheus-alertmanager
```

The exact service name depends on your Helm release name.

---

# 4. Find the Alertmanager Pod

You can use:

```bash
kubectl get pods -n monitoring | grep alertmanager
```

You should get something similar to:

```text
alertmanager-monitoring-kube-prometheus-alertmanager-0
```

---

# 5. Access Alertmanager UI

The easiest way during learning is port forwarding.

First find the service:

```bash
kubectl get svc -n monitoring | grep alertmanager
```

Suppose you see:

```text
monitoring-kube-prometheus-alertmanager
```

Then:

```bash
kubectl port-forward -n monitoring \
svc/monitoring-kube-prometheus-alertmanager 9093:9093
```

Now open:

```text
http://localhost:9093
```

You should see the **Alertmanager UI**.

---

# 6. Alertmanager UI

The important areas are things such as:

```text
Alerts
Silences
Status
```

The exact UI can vary slightly with your Alertmanager version.

Think of it as:

```text
┌──────────────────────────────────────┐
│           Alertmanager               │
├──────────────────────────────────────┤
│ Alerts                               │
│                                      │
│ [FIRING] High CPU                    │
│ [FIRING] Pod Restarting              │
│                                      │
├──────────────────────────────────────┤
│ Silences                             │
│                                      │
├──────────────────────────────────────┤
│ Status                               │
└──────────────────────────────────────┘
```

---

# 7. But Where Do Alerts Come From?

This is the critical architecture.

Alertmanager doesn't normally decide:

> CPU is greater than 80%.

**Prometheus does that.**

For example:

```text
PromQL:

CPU > 80%
```

becomes an **alerting rule**.

Then:

```text
Prometheus
   │
   │ Alert rule evaluates TRUE
   ▼
Alertmanager
```

---

# 8. Alert Rule Example

Let's create a simple alert:

> Fire an alert if a Kubernetes Pod has restarted during the last 10 minutes.

The PromQL could be:

```promql
increase(
  kube_pod_container_status_restarts_total[10m]
) > 0
```

This means:

```text
Look at restart counter
        ↓
Look at last 10 minutes
        ↓
Did it increase?
        ↓
YES → Alert
```

But we need to tell Prometheus that this is actually an **alert rule**.

---

# 9. Kubernetes Way of Creating Alert Rules

Because you're using `kube-prometheus-stack`, the recommended Kubernetes approach is to use:

```text
PrometheusRule
```

This is a Kubernetes Custom Resource.

Architecture:

```text
Kubernetes
    │
    ▼
PrometheusRule
    │
    ▼
Prometheus
    │
    ▼
Alertmanager
```

---

# 10. Create Your First Alert Rule

Create a file:

```text
pod-restart-alert.yaml
```

Put:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: kubernetes-pod-restart-alert
  namespace: monitoring
  labels:
    release: monitoring
spec:
  groups:
    - name: kubernetes-pod-alerts
      rules:
        - alert: KubernetesPodRestarting
          expr: |
            increase(
              kube_pod_container_status_restarts_total[10m]
            ) > 0
          for: 5m
          labels:
            severity: warning
          annotations:
            summary: "Kubernetes Pod is restarting"
            description: "Pod {{ $labels.namespace }}/{{ $labels.pod }} has restarted during the last 10 minutes."
```

---

# 11. Understand the Rule

Let's break it down.

### Alert name

```yaml
alert: KubernetesPodRestarting
```

This becomes the alert name.

---

### Expression

```yaml
expr: |
  increase(
    kube_pod_container_status_restarts_total[10m]
  ) > 0
```

This is the PromQL.

---

### `for`

```yaml
for: 5m
```

This is extremely important.

It means:

> The condition must remain true for 5 minutes before the alert becomes firing.

Without:

```yaml
for: 5m
```

a temporary condition could immediately trigger an alert.

Think:

```text
CPU > 80%
     │
     ├── 10 seconds → don't alert yet
     ├── 2 minutes   → don't alert yet
     ├── 4 minutes   → don't alert yet
     └── 5 minutes   → FIRE
```

---

# 12. Labels

We added:

```yaml
labels:
  severity: warning
```

This is metadata attached to the alert.

You can have:

```text
severity=info
severity=warning
severity=critical
```

For example:

```yaml
labels:
  severity: critical
```

Later Alertmanager can route:

```text
critical → PagerDuty/Teams
warning  → Slack
info     → Email
```

---

# 13. Annotations

We have:

```yaml
annotations:
  summary: "Kubernetes Pod is restarting"
```

and:

```yaml
description: "Pod {{ $labels.namespace }}/{{ $labels.pod }} has restarted during the last 10 minutes."
```

Annotations provide human-readable information.

So an alert might appear as:

```text
KubernetesPodRestarting

Severity: warning

Summary:
Kubernetes Pod is restarting

Description:
Pod production/api-123 has restarted
during the last 10 minutes.
```

---

# 14. Apply the Rule

Run:

```bash
kubectl apply -f pod-restart-alert.yaml
```

You should see:

```text
prometheusrule.monitoring.coreos.com/kubernetes-pod-restart-alert created
```

---

# 15. Verify the PrometheusRule

Run:

```bash
kubectl get prometheusrule -n monitoring
```

You should see:

```text
kubernetes-pod-restart-alert
```

You can inspect it:

```bash
kubectl describe prometheusrule \
kubernetes-pod-restart-alert \
-n monitoring
```

---

# 16. Check the Alert in Prometheus

Now navigate to your Prometheus UI:

```text
http://localhost:9090
```

Then navigate:

```text
Alerts
```

You should eventually see:

```text
KubernetesPodRestarting
```

The state might be:

```text
Inactive
```

or:

```text
Pending
```

or:

```text
Firing
```

---

# 17. Understand Alert States

This is extremely important.

```text
               Condition false
                     │
                     ▼
                 INACTIVE
                     │
                     │ condition becomes true
                     ▼
                  PENDING
                     │
                     │ remains true for `for`
                     ▼
                  FIRING
```

For our rule:

```yaml
for: 5m
```

means:

```text
Condition TRUE
      ↓
Pending
      ↓
5 minutes
      ↓
Firing
```

---

# 18. What Happens When It Fires?

Once Prometheus determines:

```text
KubernetesPodRestarting = FIRING
```

Prometheus sends the alert to Alertmanager.

Architecture:

```text
PromQL
  │
  ▼
Expression TRUE
  │
  ▼
Pending
  │
  ▼
Firing
  │
  ▼
Prometheus
  │
  │ HTTP
  ▼
Alertmanager
```

Now Alertmanager takes over.

---

# 19. Check Alertmanager

Go back to:

```text
http://localhost:9093
```

Navigate:

```text
Alerts
```

You should see the alert if it is firing.

Something conceptually like:

```text
┌─────────────────────────────────────────┐
│ KubernetesPodRestarting                 │
│                                         │
│ severity: warning                       │
│ namespace: production                   │
│ pod: api-123                            │
│                                         │
│ Pod production/api-123 has restarted    │
└─────────────────────────────────────────┘
```

---

# 20. This Is Where Alertmanager Becomes Powerful

Suppose Prometheus generates:

```text
100 alerts
```

You don't necessarily want:

```text
100 Slack messages
```

Alertmanager can group them.

For example:

```text
100 Pod alerts
       ↓
Alertmanager
       ↓
1 notification
```

Something like:

```text
Production Kubernetes Alerts

12 Pods restarting
5 Pods not ready
3 nodes with high CPU
```

This is called **grouping**.

---

# 21. Routing

You can also route based on labels.

Remember:

```yaml
labels:
  severity: warning
```

Alertmanager can make decisions based on that.

Conceptually:

```text
                    Alertmanager
                         │
             ┌───────────┴───────────┐
             │                       │
       severity=warning       severity=critical
             │                       │
             ▼                       ▼
           Slack                   Email
```

This is called **routing**.

---

# 22. Receivers

A receiver defines where the notification goes.

Examples:

```text
Email
Slack
Microsoft Teams
Webhook
PagerDuty
Opsgenie
```

Conceptually:

```yaml
receivers:
  - name: slack
  - name: email
  - name: webhook
```

Then routing determines which receiver gets which alert.

---

# 23. Silence

Suppose you're doing maintenance.

You know:

```text
Node worker-1
```

will be offline for one hour.

You don't want:

```text
CPU alert
Node down alert
Pod alert
Network alert
...
```

during maintenance.

You can create a **Silence** in Alertmanager.

Navigate:

```text
Alertmanager
   ↓
Silences
   ↓
New Silence
```

You specify matchers such as:

```text
alertname = KubernetesPodRestarting
```

and a duration.

During that period, matching notifications are suppressed.

---

# 24. Silencing Does NOT Stop Prometheus

This is an important distinction.

If you silence an alert:

```text
Prometheus
   │
   │ continues evaluating
   ▼
Alert is still firing
   │
   ▼
Alertmanager
   │
   │ SILENCE
   ▼
No notification
```

So:

```text
Silence ≠ disable monitoring
```

It only suppresses notifications.

---

# 25. Inhibition

Another Alertmanager feature is **inhibition**.

Suppose:

```text
Node Down
```

causes:

```text
50 Pods Down
```

You don't necessarily want 50 individual notifications.

You could configure:

```text
NodeDown
   ↓
inhibit
   ↓
PodDown
```

Meaning:

> If the node is down, suppress lower-level Pod alerts caused by that node failure.

This helps prevent alert storms.

---

# 26. Alertmanager vs Grafana Alerting

You'll eventually encounter both:

```text
Prometheus Alerting
        +
Alertmanager
```

and:

```text
Grafana Alerting
```

For the learning path we're following, I recommend first mastering:

```text
Prometheus
   ↓
PrometheusRule
   ↓
Alertmanager
```

because this is very common in Kubernetes environments using `kube-prometheus-stack`.

Later we'll compare Grafana Alerting with Prometheus Alertmanager.

---

# 27. Our Complete Monitoring Architecture

You should now understand the entire flow:

```text
                         Kubernetes
                             │
             ┌───────────────┼───────────────┐
             │               │               │
             ▼               ▼               ▼
           Nodes            Pods        Applications
             │               │               │
             └───────────────┼───────────────┘
                             ▼
                        Prometheus
                             │
                       PromQL / Rules
                             │
                ┌────────────┴────────────┐
                │                         │
                ▼                         ▼
             Grafana                 Alertmanager
                │                         │
                ▼                ┌────────┼────────┐
           Dashboards             ▼        ▼        ▼
                                Email    Slack    Webhook
```

This is the core of Kubernetes observability.

---

# 28. Your First Real Alert Lab

Let's make a slightly better alert.

Create:

```text
pod-restart-alert.yaml
```

with:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: kubernetes-pod-restart-alert
  namespace: monitoring
  labels:
    release: monitoring
spec:
  groups:
    - name: kubernetes-pod-alerts
      rules:

        - alert: KubernetesPodRestarting
          expr: |
            increase(
              kube_pod_container_status_restarts_total[10m]
            ) > 0
          for: 5m
          labels:
            severity: warning
          annotations:
            summary: "Kubernetes Pod is restarting"
            description: "Pod {{ $labels.namespace }}/{{ $labels.pod }} has restarted during the last 10 minutes."
```

Apply:

```bash
kubectl apply -f pod-restart-alert.yaml
```

Then check:

```bash
kubectl get prometheusrule -n monitoring
```

Then:

```text
Prometheus
 → Alerts
```

and:

```text
Alertmanager
 → Alerts
```

---

# 29. Don't Worry About Slack/Email Yet

I recommend **not configuring Slack or email yet**.

First make sure this pipeline works:

```text
Metric
 ↓
PromQL
 ↓
PrometheusRule
 ↓
Pending
 ↓
Firing
 ↓
Alertmanager
```

Once that works, we'll configure a real receiver.

Otherwise, if something doesn't work, there are too many possible places to troubleshoot.

---

# 30. Your Learning Roadmap From Here

We've now reached:

```text
Lesson 1  Monitoring Fundamentals
Lesson 2  Prometheus Architecture
Lesson 3  Kubernetes + Helm Installation
Lesson 4  Prometheus Components
Lesson 5  Targets & Scraping
Lesson 6  Metrics & Labels
Lesson 7  PromQL Fundamentals
Lesson 8  PromQL Deep Dive
Lesson 9  Kubernetes PromQL Filtering
Lesson 10 Grafana Fundamentals
Lesson 11 Grafana Dashboard
Lesson 12 Grafana Variables
Lesson 13 Dashboard Formatting
Lesson 14 Alertmanager        ← YOU ARE HERE
```

Next we'll do:

# Lesson 15 — Real Kubernetes Alert Rules

We'll create a proper alert set:

```text
Kubernetes Alerts
│
├── Node Down
├── High Node CPU
├── High Node Memory
├── Pod Restarting
├── Pod Not Ready
├── Deployment Replicas Unavailable
├── High CPU Pod
└── High Memory Pod
```

We'll also learn the difference between:

```text
warning
critical
```

and how `for:` prevents false alarms.

Then:

# Lesson 16 — Alertmanager Routing

We'll configure:

```text
severity=warning
       ↓
     Slack

severity=critical
       ↓
     Email
```

and learn:

```text
route
receiver
group_by
group_wait
group_interval
repeat_interval
silence
inhibition
```

Finally, we'll connect the whole thing to your original goal:

```text
GitLab CI/CD
      ↓
Deploy application to Kubernetes
      ↓
Prometheus monitors it
      ↓
Grafana visualizes it
      ↓
Alertmanager alerts on problems
      ↓
GitLab pipeline + monitoring
```

That's where the **Prometheus + Grafana + Kubernetes + GitLab CI/CD** learning path comes together.
