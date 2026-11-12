# Alert Management — Lesson 3

## Alertmanager Notifications 📢

Now we move one step forward.

In Lesson 2, we learned:

```text
PrometheusRule
      ↓
Prometheus
      ↓
Alert FIRING
      ↓
Alertmanager
```

Now we want:

```text
Alert FIRING
      ↓
Alertmanager
      ↓
Route
      ↓
Receiver
      ↓
Notification
```

Alertmanager is responsible for routing alerts to notification integrations such as email, Slack, PagerDuty, webhooks, etc. ([Prometheus][1])

---

# 1. What is a Receiver?

Think of a **receiver as the destination**.

For example:

```text
receiver = email
```

means:

> Send the alert to an email address.

Or:

```text
receiver = slack
```

means:

> Send the alert to Slack.

Or:

```text
receiver = webhook
```

means:

> Send the alert to an HTTP endpoint.

So:

```text
Alert
  ↓
Alertmanager
  ↓
Receiver
  ↓
Email / Slack / Webhook
```

---

# 2. What is a Route?

A **route decides which receiver should get the alert**.

For example, suppose we have:

```text
severity=warning
```

and

```text
severity=critical
```

We could configure:

```text
WARNING
   ↓
DevOps Team

CRITICAL
   ↓
DevOps Team + On-call Team
```

Alertmanager's routing tree uses alert labels to determine which receiver handles an alert. ([Prometheus][2])

---

# 3. Simple Example

Imagine our alert is:

```yaml
labels:
  severity: warning
```

Alertmanager receives:

```text
TestKubernetesAlert
severity=warning
```

The routing logic could be:

```text
                    Alertmanager
                         |
                 severity = warning?
                    /          \
                  YES            NO
                   |              |
              warning-team     default
                   |              |
                 Email          Email
```

---

# 4. Important Alertmanager Configuration

A simplified Alertmanager configuration looks like this:

```yaml
route:
  receiver: default

receivers:

  - name: default

  - name: warning-team
```

There are two concepts here:

### Route

```yaml
route:
  receiver: default
```

### Receivers

```yaml
receivers:

  - name: default

  - name: warning-team
```

The names must match.

For example:

```yaml
receiver: warning-team
```

must have:

```yaml
- name: warning-team
```

---

# 5. How It Fits With Our Alert

Our alert contains:

```yaml
labels:
  severity: warning
```

We could create a route:

```yaml
route:
  receiver: default

  routes:
    - matchers:
        - severity="warning"
      receiver: warning-team
```

Now the flow becomes:

```text
TestKubernetesAlert
severity=warning
        ↓
   Alertmanager
        ↓
 severity="warning"
        ↓
   warning-team
        ↓
   Notification
```

This is the basic idea of **alert routing**.

---

# 6. But We Are NOT Going to Change Your Configuration Yet

Because you're using **kube-prometheus-stack**, Alertmanager configuration is managed through Kubernetes/Helm rather than simply editing a local `alertmanager.yml`.

So before we configure email or Slack, let's understand your current setup.

Run:

```bash
kubectl get alertmanager -n monitoring
```

You should see something like:

```text
NAME
monitoring-kube-prometheus-alertmanager
```

Then run:

```bash
kubectl get alertmanager -n monitoring -o yaml
```

Don't modify anything yet.

---

# 7. Our Learning Sequence

We'll do this progressively:

### Lesson 3 — Part A

Understand:

```text
Receiver
Route
Notification
```

### Lesson 3 — Part B

Configure a **simple test receiver**.

We'll use a webhook first because it doesn't require SMTP/email configuration.

```text
Prometheus
    ↓
Alert
    ↓
Alertmanager
    ↓
Webhook
    ↓
See notification
```

Prometheus's own alerting tutorial uses a webhook receiver as a simple notification example. ([Prometheus][3])

### Lesson 4

Routing:

```text
severity=warning
        ↓
Warning Receiver

severity=critical
        ↓
Critical Receiver
```

### Lesson 5

Grouping:

```text
100 alerts
    ↓
Alertmanager
    ↓
1 grouped notification
```

### Lesson 6

Silencing:

```text
Maintenance
     ↓
Silence alerts
     ↓
No notifications
```

### Lesson 7

Inhibition:

```text
NodeDown
   ↓
Suppress related PodDown alerts
```

---

## Your next step

Run **only this command**:

```bash
kubectl get alertmanager -n monitoring
```

Send me the output.

Then we'll configure your **first real Alertmanager notification** without changing anything unnecessarily.

[1]: https://prometheus.io/docs/alerting/latest/alertmanager/?utm_source=chatgpt.com "Alertmanager | Prometheus"
[2]: https://prometheus.io/docs/alerting/latest/configuration/?utm_source=chatgpt.com "Configuration | Prometheus"
[3]: https://prometheus.io/docs/tutorials/alerting_based_on_metrics/?utm_source=chatgpt.com "Alerting based on metrics | Prometheus"
