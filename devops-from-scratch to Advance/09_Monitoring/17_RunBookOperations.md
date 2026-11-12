Absolutely. Let's start the **Operations & Runbooks** topic from the beginning, keeping it as simple and hands-on as we did with Alert Management.

# Operations & Runbooks — Lesson 1

## Incident Response Basics

The goal is to learn what happens **after an alert is generated**.

So far we have:

```text
Application / Kubernetes
        ↓
     Metrics
        ↓
    Prometheus
        ↓
      Alert
        ↓
   Alertmanager
        ↓
 Notification
```

Now we add the operations process:

```text
Alert
  ↓
Incident
  ↓
Investigate
  ↓
Identify root cause
  ↓
Take corrective action
  ↓
Verify
  ↓
Close incident
  ↓
Document / Improve
```

---

# 1. What is an Incident?

An **incident** is a problem that affects a system, application, deployment, security, or users and requires action.

For example:

```text
Deployment
    ↓
New version deployed
    ↓
Pods start crashing
    ↓
Prometheus detects restarts
    ↓
Alert fires
    ↓
DevOps engineer investigates
```

The alert is **not the incident itself**.

Think:

> **Alert = something needs attention**

> **Incident = a problem requiring investigation/action**

---

# 2. Simple Example

Suppose you deploy:

```text
myapp:v2
```

After deployment:

```text
Pod
 ↓
CrashLoopBackOff
 ↓
Restart count increases
 ↓
Prometheus
 ↓
PodRestarting alert
 ↓
Alertmanager
 ↓
DevOps engineer
```

Now the engineer has to answer:

### What happened?

```text
Why is the Pod crashing?
```

### What changed?

```text
Was a new version deployed?
```

### What is the impact?

```text
Are users affected?
```

### What should we do?

```text
Fix deployment?
Rollback?
Scale?
Restart?
```

---

# 3. Incident Response Lifecycle

Remember this simple lifecycle:

```text
        ┌──────────────┐
        │    Detect    │
        └──────┬───────┘
               ↓
        ┌──────────────┐
        │    Triage    │
        └──────┬───────┘
               ↓
        ┌──────────────┐
        │ Investigate  │
        └──────┬───────┘
               ↓
        ┌──────────────┐
        │   Correct    │
        └──────┬───────┘
               ↓
        ┌──────────────┐
        │   Verify     │
        └──────┬───────┘
               ↓
        ┌──────────────┐
        │    Close     │
        └──────────────┘
```

Let's understand each one.

---

# 4. Detect

Something tells us there is a problem.

Examples:

```text
Prometheus Alert
Grafana Alert
GitLab CI failure
Security scan failure
User reports
```

In our environment:

```text
Prometheus
    ↓
PodRestarting
    ↓
Alertmanager
```

---

# 5. Triage

**Triage means quickly determining how serious the problem is.**

Ask:

```text
Is production affected?
How many services?
How many users?
Is this critical?
```

For example:

### Warning

```text
One Pod restarted once
```

Maybe investigate but no immediate emergency.

### Critical

```text
Production application has zero healthy replicas
```

Immediate action required.

---

# 6. Investigate

Now we find the cause.

For Kubernetes, our first commands might be:

```bash
kubectl get pods -n <namespace>
```

Then:

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Then:

```bash
kubectl logs <pod-name> -n <namespace>
```

For deployments:

```bash
kubectl get deployment -n <namespace>
```

```bash
kubectl describe deployment <deployment-name> -n <namespace>
```

And we can look at Prometheus/Grafana metrics.

---

# 7. Correct

Once we understand the problem, we take action.

Examples:

```text
Bad configuration
      ↓
Fix configuration
```

or:

```text
Bad deployment
      ↓
Rollback
```

or:

```text
Insufficient capacity
      ↓
Scale application
```

---

# 8. Verify

This step is extremely important.

Don't just fix something and say:

> "Done."

Check whether the system actually recovered.

For Kubernetes:

```bash
kubectl get pods -n <namespace>
```

Check:

```text
READY
STATUS
RESTARTS
```

Then check Grafana/Prometheus:

```text
CPU
Memory
Pod availability
Error rate
Alerts
```

And finally:

```text
Alert → Resolved
```

---

# 9. Close

Once the system is healthy:

```text
Incident
   ↓
Resolved
   ↓
Close
```

But ideally we also record:

```text
What happened?
Why did it happen?
What fixed it?
How can we prevent it?
```

That becomes our **incident record/runbook improvement**.

---

# 10. What is a Runbook?

A **runbook is a predefined set of steps for handling a known operational problem.**

Instead of someone asking:

> "What do I do when a deployment fails?"

we have:

```text
RUNBOOK: Failed Deployment

1. Check deployment
2. Check Pods
3. Check events
4. Check logs
5. Identify failure
6. Roll back if required
7. Verify recovery
8. Record incident
```

This makes operations more consistent.

---

# 11. Our First Runbook

We're going to create:

# `RUNBOOK-001: Failed Kubernetes Deployment`

Scenario:

```text
Developer
    ↓
GitLab CI/CD
    ↓
Deploy new version
    ↓
Kubernetes
    ↓
Pods fail
    ↓
Alert
    ↓
DevOps investigates
```

Our runbook will eventually look like:

```text
RUNBOOK-001
Failed Kubernetes Deployment
─────────────────────────────

1. Detect
2. Check deployment
3. Check pods
4. Check events
5. Check logs
6. Determine cause
7. Rollback if necessary
8. Verify application
9. Close incident
10. Document corrective action
```

---

# 12. Hands-on — Let's Simulate an Incident

Rather than just reading the runbook, we'll **create a controlled failure in Kubernetes**.

We'll deploy a deliberately broken application.

For example:

```yaml
image: nginx:does-not-exist
```

Kubernetes will try to pull the image:

```text
nginx:does-not-exist
        ↓
ImagePullBackOff
```

Then we'll investigate it exactly like a real incident.

The flow will be:

```text
Broken Deployment
      ↓
Pod failure
      ↓
kubectl investigation
      ↓
Find root cause
      ↓
Fix
      ↓
Verify
```

This is much more useful than memorizing commands.

---

## Lesson 1 Exercise — Create the Broken Deployment

Create:

```text
failed-deployment.yaml
```

with:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: broken-app
  namespace: default

spec:
  replicas: 2

  selector:
    matchLabels:
      app: broken-app

  template:
    metadata:
      labels:
        app: broken-app

    spec:
      containers:
        - name: app
          image: nginx:does-not-exist
          ports:
            - containerPort: 80
```

Apply it:

```bash
kubectl apply -f failed-deployment.yaml
```

Then:

```bash
kubectl get pods
```

You should eventually see something similar to:

```text
NAME                          READY   STATUS
broken-app-xxxxx-xxxxx        0/1     ImagePullBackOff
broken-app-xxxxx-xxxxx        0/1     ImagePullBackOff
```

**Stop there. Don't fix it yet.**

The next part of the lesson will be:

> **Incident Investigation — How do we go from `ImagePullBackOff` to the actual root cause?**

We'll use only a few commands:

```bash
kubectl get pods
kubectl describe pod
kubectl logs
kubectl get events
```

and turn those steps into our first proper **Kubernetes troubleshooting runbook**.
