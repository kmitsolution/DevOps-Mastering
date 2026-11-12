# Alert Management — Lesson 2

## Hands-on: Create Your First Kubernetes Alert 🚨

In this lesson, we will create **one simple alert**, make it fire, and see it in both **Prometheus** and **Alertmanager**.

### What we will build

```text
PrometheusRule
      ↓
 Prometheus
      ↓
 Alert: TestKubernetesAlert
      ↓
 Alertmanager
```

We are **not doing email/Slack yet**. First understand the basic flow.

---

## Step 1 — Check Alertmanager

Run:

```bash
kubectl get pods -n monitoring
```

You should see something similar to:

```text
alertmanager-monitoring-kube-prometheus-alertmanager-0
grafana-xxxxx
prometheus-monitoring-kube-prometheus-prometheus-0
```

Now check services:

```bash
kubectl get svc -n monitoring
```

Look for the Alertmanager service.

You can also directly search:

```bash
kubectl get svc -n monitoring | grep alertmanager
```

---

# Step 2 — Open Alertmanager

Suppose your service is:

```text
monitoring-kube-prometheus-alertmanager
```

Run:

```bash
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-alertmanager 9093:9093
```

You should get:

```text
Forwarding from 127.0.0.1:9093 -> 9093
```

Open:

```text
http://localhost:9093
```

You should see the **Alertmanager UI**.

Keep this terminal running.

---

# Step 3 — Create a Test Alert

Create a file:

```text
test-alert.yaml
```

Put this inside:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule

metadata:
  name: test-alert
  namespace: monitoring
  labels:
    release: monitoring

spec:
  groups:
    - name: test-alerts

      rules:
        - alert: TestKubernetesAlert

          expr: vector(1)

          for: 1m

          labels:
            severity: warning

          annotations:
            summary: "Test alert"
            description: "This is a test Kubernetes alert."
```

---

# Step 4 — Understand the Important Parts

### `alert`

```yaml
alert: TestKubernetesAlert
```

This is the **name of our alert**.

---

### `expr`

```yaml
expr: vector(1)
```

This is our PromQL condition.

`vector(1)` always returns:

```text
1
```

So:

```text
condition = TRUE
```

all the time.

This is useful for testing.

---

### `for`

```yaml
for: 1m
```

This means:

> The condition must remain true for 1 minute before the alert becomes FIRING.

So:

```text
Condition TRUE
      ↓
   Pending
      ↓ 1 minute
   Firing
```

---

### `severity`

```yaml
severity: warning
```

We are adding a label to tell Alertmanager:

> This is a warning-level alert.

Later we'll use this for routing.

---

### `annotations`

```yaml
annotations:
  summary: "Test alert"
  description: "This is a test Kubernetes alert."
```

These contain the **human-readable information** about the alert.

Later, when we send email/Slack notifications, these values become very useful.

---

# Step 5 — Apply the Alert

Run:

```bash
kubectl apply -f test-alert.yaml
```

Expected:

```text
prometheusrule.monitoring.coreos.com/test-alert created
```

Verify:

```bash
kubectl get prometheusrule -n monitoring
```

You should see:

```text
NAME
test-alert
```

---

# Step 6 — Check Prometheus

If Prometheus is not already accessible, find its service:

```bash
kubectl get svc -n monitoring | grep prometheus
```

Then port-forward the Prometheus service.

For example:

```bash
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-prometheus 9090:9090
```

Open:

```text
http://localhost:9090
```

Now go to:

**Prometheus → Alerts**

You should see:

```text
TestKubernetesAlert
```

Initially you may see:

```text
PENDING
```

After approximately one minute:

```text
FIRING
```

---

# Step 7 — Understand What Just Happened

This is the most important concept.

Our expression:

```promql
vector(1)
```

is always true.

Therefore:

```text
vector(1)
    ↓
TRUE
    ↓
Prometheus evaluates alert
    ↓
PENDING
    ↓
1 minute
    ↓
FIRING
```

---

# Step 8 — Check Alertmanager

Now open:

```text
http://localhost:9093
```

Go to:

**Alertmanager → Alerts**

You should see:

```text
TestKubernetesAlert
```

with:

```text
Severity: warning
```

So now we have successfully created:

```text
             Kubernetes
                  │
                  │
            PrometheusRule
                  │
                  ↓
             Prometheus
                  │
             Alert fires
                  │
                  ↓
             Alertmanager
                  │
                  ↓
              Alert UI
```

🎯 **This is the basic Alert Management flow.**

---

# Step 9 — Delete the Test Alert

After testing:

```bash
kubectl delete prometheusrule test-alert -n monitoring
```

Verify:

```bash
kubectl get prometheusrule -n monitoring
```

The test alert should disappear.

---

## One thing to remember

There are **three different concepts**:

| Component    | Responsibility                  |
| ------------ | ------------------------------- |
| PromQL       | Defines the condition           |
| Prometheus   | Evaluates the condition         |
| Alertmanager | Manages and sends notifications |

For example:

```text
CPU > 80%
   ↓
PromQL condition

Prometheus
   ↓
Is CPU > 80% for 5 minutes?
   ↓
YES
   ↓
Alert = FIRING

Alertmanager
   ↓
Who should receive it?
   ↓
Email / Slack / Teams
```

### Next lesson

**Lesson 3 — Alertmanager Notifications**

We'll create a simple **receiver** and understand:

```text
Alert
  ↓
Alertmanager
  ↓
Route
  ↓
Receiver
  ↓
Notification
```

Then we'll move to **routing by severity: warning vs critical**.
