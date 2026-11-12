Absolutely. Let's **reset Lesson 3** and do it slowly. We won't mix Alertmanager, Flannel, webhooks, and configuration all at once.

# Alert Management — Lesson 3

## Alertmanager Notifications

### Goal of this lesson

At the end, you should understand this:

```text
Prometheus
    ↓
Alert fires
    ↓
Alertmanager
    ↓
Receiver
    ↓
Notification
```

We will do **one small practical example**.

---

# Part 1 — First understand the concepts

## 1. Prometheus creates the alert

Suppose we have:

```yaml
- alert: HighCPU
  expr: cpu_usage > 80
  for: 5m
```

Prometheus checks:

```text
CPU > 80% ?
```

If yes for 5 minutes:

```text
HighCPU = FIRING
```

---

## 2. Alertmanager receives the alert

The flow is:

```text
Prometheus
     │
     │ HighCPU = FIRING
     ↓
Alertmanager
```

Alertmanager now asks:

> What should I do with this alert?

---

## 3. Receiver

A **receiver** means:

> Where should I send the notification?

Examples:

```text
Email
Slack
Microsoft Teams
PagerDuty
Webhook
```

For example:

```text
receiver = email
```

means:

```text
Alertmanager
      ↓
Email
```

---

## 4. Route

A **route** means:

> Which receiver should receive this alert?

For example:

```text
severity=warning
       ↓
Warning receiver

severity=critical
       ↓
Critical receiver
```

So:

```text
                  Alertmanager
                       │
             ┌─────────┴─────────┐
             ↓                   ↓
       severity=warning    severity=critical
             ↓                   ↓
       Warning receiver    Critical receiver
```

That's **routing**.

---

# Part 2 — Your current Kubernetes setup

You already have:

```text
kube-prometheus-stack
        ↓
Prometheus
        ↓
Alertmanager
```

And your Alertmanager Pod is currently:

```text
2/2 Running
```

Your logs also showed that Alertmanager successfully loaded its configuration. 

So we don't need to reinstall anything.

---

# Part 3 — Forget the webhook for now

This is where I made things unnecessarily complicated earlier.

**Don't create `webhook.yaml`.**

Don't create:

```text
webhook.py
webhook-test
```

Don't change Flannel.

For this lesson, we first learn **Alertmanager routing and receivers**.

---

# Part 4 — See Alertmanager

First make Alertmanager accessible.

Run:

```bash
kubectl get svc -n monitoring | grep alertmanager
```

You'll get a service name similar to:

```text
monitoring-kube-prometheus-alertmanager
```

Then:

```bash
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-alertmanager 9093:9093
```

You should see:

```text
Forwarding from 127.0.0.1:9093 -> 9093
```

Open your browser:

```text
http://localhost:9093
```

---

# Part 5 — Look at Alertmanager UI

You should see the Alertmanager interface.

At this point, **don't configure anything**.

Just understand:

```text
Alertmanager UI
```

is where you can see alerts that Prometheus has sent to Alertmanager.

---

# Part 6 — Let's create one test alert

Now we need an alert.

Create:

```text
test-alert.yaml
```

Use this:

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
            summary: "Test Kubernetes alert"
            description: "This is a test alert."
```

Apply:

```bash
kubectl apply -f test-alert.yaml
```

---

# Part 7 — Check Prometheus

Open:

```text
http://localhost:9090
```

Then navigate:

**Prometheus → Alerts**

Search for:

```text
TestKubernetesAlert
```

You should eventually see:

```text
TestKubernetesAlert
```

with:

```text
FIRING
```

If you don't see it, **stop here** and we'll fix the PrometheusRule before continuing.

---

# Part 8 — Check Alertmanager

Now go to:

```text
http://localhost:9093
```

Navigate to:

**Alerts**

You should see:

```text
TestKubernetesAlert
```

with:

```text
severity = warning
```

This proves:

```text
Prometheus
    ↓
TestKubernetesAlert
    ↓
Alertmanager
    ↓
Alert visible in Alertmanager
```

---

# Part 9 — Now understand Notification

Currently we only have:

```text
Prometheus
    ↓
Alert
    ↓
Alertmanager
    ↓
Alertmanager UI
```

But nobody has received an email.

Why?

Because we haven't configured a **receiver**.

We eventually want:

```text
Prometheus
    ↓
Alert
    ↓
Alertmanager
    ↓
Receiver
    ↓
Email / Slack / Teams
```

---

# Part 10 — Our next practical step

Now that the concept is clear, we'll configure a receiver.

We'll do it in this order:

### 3.1 Receiver

```text
Alertmanager
      ↓
Receiver
```

### 3.2 Route

```text
severity=warning
       ↓
Warning receiver
```

### 3.3 Notification

```text
Warning receiver
       ↓
Webhook
```

### 3.4 Test

```text
TestKubernetesAlert
       ↓
FIRING
       ↓
Alertmanager
       ↓
Route
       ↓
Receiver
       ↓
Notification
```

---

## 🚦 For now, do only this

Let's make sure the **basic alert works first**.

Run:

```bash
kubectl apply -f test-alert.yaml
```

Then check:

**Prometheus → Alerts → `TestKubernetesAlert`**

and tell me whether you see:

```text
FIRING
```

Once that works, I'll give you **Lesson 3.1: Create the first Receiver**, with just **one step at a time**.
