Yes. Let's keep this section **strictly to one topic: Runbook for Failed Deployment**.

We will leave security alerts, rollbacks, Issues, and MRs for later.

# Operations & Runbooks

## Runbook 1 — Failed Kubernetes Deployment

### Objective

When a Kubernetes deployment fails, we should have a **standard sequence of steps** instead of randomly running commands.

Our runbook:

```text
Failed Deployment
       ↓
1. Detect
       ↓
2. Check Deployment
       ↓
3. Check Pods
       ↓
4. Check Events
       ↓
5. Check Logs
       ↓
6. Identify Root Cause
       ↓
7. Fix
       ↓
8. Verify
       ↓
9. Close
```

---

# Step 1 — Detect the failed deployment

A deployment failure can be detected from:

* GitLab CI/CD
* Grafana
* Prometheus/Alertmanager
* Kubernetes itself

For this exercise, we'll assume:

> **A new deployment was made and the application is not running.**

---

# Step 2 — Check the Deployment

First command:

```bash
kubectl get deployments
```

Example:

```text
NAME         READY   UP-TO-DATE   AVAILABLE
broken-app   0/2     2            0
```

This immediately tells us:

```text
Desired replicas:     2
Ready replicas:       0
Available replicas:   0
```

So something is wrong.

---

# Step 3 — Check the Pods

Now:

```bash
kubectl get pods
```

Suppose we get:

```text
NAME                          READY   STATUS
broken-app-7f8d9c-x1abc       0/1     ImagePullBackOff
broken-app-7f8d9c-x2def       0/1     ImagePullBackOff
```

Now we know the problem is at the **Pod level**.

The status:

```text
ImagePullBackOff
```

is already giving us a clue.

---

# Step 4 — Check Pod Details

Now we investigate instead of guessing.

Take one Pod:

```bash
kubectl describe pod broken-app-7f8d9c-x1abc
```

Look at the bottom:

```text
Events:
```

You might see:

```text
Failed to pull image "nginx:does-not-exist"
```

and:

```text
manifest unknown
```

Now we have a much stronger indication of the root cause.

---

# Step 5 — Check Events

We can also look at recent Kubernetes events:

```bash
kubectl get events --sort-by=.lastTimestamp
```

You might see:

```text
Failed to pull image
Back-off pulling image
```

This confirms Kubernetes cannot retrieve the container image.

---

# Step 6 — Check Logs

Normally we would also check:

```bash
kubectl logs <pod-name>
```

But there is an important point:

If the image **cannot even be pulled**, the container never starts.

Therefore:

```text
ImagePullBackOff
      ↓
Container never started
      ↓
No application logs
```

So `kubectl logs` may not provide anything useful in this particular failure.

This is why understanding the Pod status and Events is important.

---

# Step 7 — Identify Root Cause

Our investigation produced:

```text
Deployment
    ↓
Pods
    ↓
ImagePullBackOff
    ↓
describe pod
    ↓
Failed to pull image
    ↓
nginx:does-not-exist
```

### Root Cause

```text
Invalid/non-existent container image:
nginx:does-not-exist
```

This is much better than saying:

> "Kubernetes deployment failed."

We now know **why** it failed.

---

# Step 8 — Fix the Deployment

Change:

```yaml
image: nginx:does-not-exist
```

to:

```yaml
image: nginx:latest
```

Then apply:

```bash
kubectl apply -f failed-deployment.yaml
```

Watch the Pods:

```bash
kubectl get pods -w
```

Eventually:

```text
broken-app-xxxxx   1/1   Running
broken-app-xxxxx   1/1   Running
```

Press:

```text
Ctrl+C
```

to stop watching.

---

# Step 9 — Verify the Deployment

Now:

```bash
kubectl get deployment broken-app
```

We want:

```text
READY   UP-TO-DATE   AVAILABLE
2/2     2            2
```

Then:

```bash
kubectl get pods
```

We want:

```text
1/1   Running
```

And finally check the application if it has a Service/endpoint.

---

# Step 10 — Runbook Summary

Our first real runbook can now be written as:

## RUNBOOK: Failed Kubernetes Deployment

### Symptoms

```text
Deployment not ready
Pods not running
```

### Procedure

**1. Check Deployment**

```bash
kubectl get deployments
```

**2. Check Pods**

```bash
kubectl get pods
```

**3. Describe failed Pod**

```bash
kubectl describe pod <pod-name>
```

**4. Check Events**

```bash
kubectl get events --sort-by=.lastTimestamp
```

**5. Check logs**

```bash
kubectl logs <pod-name>
```

**6. Identify root cause**

Examples:

```text
ImagePullBackOff
CrashLoopBackOff
Pending
OOMKilled
CreateContainerConfigError
```

**7. Correct the problem**

For example:

```text
Wrong image
→ Correct image
```

**8. Verify**

```bash
kubectl get deployment
kubectl get pods
```

**9. Confirm application health**

Check Grafana/application endpoint/health check.

---

# The Important DevOps Pattern

The runbook isn't just a list of commands.

It is:

```text
Symptom
   ↓
Investigation
   ↓
Evidence
   ↓
Root Cause
   ↓
Corrective Action
   ↓
Verification
```

For our example:

```text
Deployment failed
       ↓
Pod = ImagePullBackOff
       ↓
describe pod
       ↓
Image does not exist
       ↓
Change image
       ↓
Pod = Running
       ↓
Deployment = 2/2
       ↓
Resolved
```

### This is the first runbook we should master.

Next, we can make this **more realistic by connecting the failed deployment to GitLab CI/CD**—so the runbook starts with **"GitLab deployment job failed"** and then moves into Kubernetes investigation. That will directly connect this topic to your DevOps workflow.
