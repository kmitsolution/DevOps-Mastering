# Operations & Runbooks — Lesson 5

## Build a Production-Style Failed Deployment Runbook

Now let's turn what we learned into a **real runbook document** that a DevOps engineer could follow during an incident.

We will still focus **only on failed deployments**.

---

# 1. What is the purpose of a Runbook?

A runbook answers:

> **"The deployment has failed. What exactly should I do?"**

Instead of:

```text
Deployment failed 😨
   ↓
Try random commands
```

we want:

```text
Deployment failed
      ↓
Follow Runbook
      ↓
Investigate
      ↓
Fix / Rollback
      ↓
Verify
```

---

# 2. RUNBOOK: Failed Kubernetes Deployment

## Runbook ID

```text
RB-K8S-001
```

## Title

```text
Failed Kubernetes Deployment
```

## Purpose

Provide a standard procedure for investigating and resolving a failed Kubernetes deployment.

---

# 3. Symptoms

You may encounter one or more of these:

```text
GitLab deployment job failed
Pods are not Ready
Deployment has unavailable replicas
CrashLoopBackOff
ImagePullBackOff
Pending
OOMKilled
Readiness probe failure
```

For example:

```bash
kubectl get pods
```

returns:

```text
NAME                         READY   STATUS
demo-app-xxx                 0/1     CrashLoopBackOff
demo-app-yyy                 0/1     CrashLoopBackOff
```

---

# 4. Impact

Before changing anything, determine the impact.

Ask:

```text
Is this production?
How many replicas are affected?
Is the application available?
Are users affected?
Is this a complete outage or partial outage?
```

Example:

```text
Application: Payment API
Desired replicas: 3
Available replicas: 0
Impact: Production outage
Severity: Critical
```

This determines whether you should **fix forward** or **rollback immediately**.

---

# 5. Prerequisites

The engineer should have:

```text
kubectl access
Kubernetes namespace access
GitLab project access
Grafana/Prometheus access
Permission to rollback deployments
```

---

# 6. Investigation Procedure

## Step 1 — Check Deployment

```bash
kubectl get deployment <deployment-name> -n <namespace>
```

Example:

```bash
kubectl get deployment demo-app -n default
```

Check:

```text
READY
UP-TO-DATE
AVAILABLE
```

---

## Step 2 — Check Pods

```bash
kubectl get pods -n <namespace>
```

Look for:

```text
CrashLoopBackOff
ImagePullBackOff
Pending
Error
OOMKilled
```

---

## Step 3 — Describe the Pod

```bash
kubectl describe pod <pod-name> -n <namespace>
```

The most important section is:

```text
Events:
```

For example:

```text
Failed to pull image
```

or:

```text
Back-off restarting failed container
```

---

# 7. Step 4 — Check Logs

If the container started:

```bash
kubectl logs <pod-name> -n <namespace>
```

If the Pod has multiple containers:

```bash
kubectl logs <pod-name> -c <container-name> -n <namespace>
```

If the container is restarting:

```bash
kubectl logs <pod-name> --previous -n <namespace>
```

That last command is particularly useful for:

```text
CrashLoopBackOff
```

because it shows logs from the previous container instance.

---

# 8. Step 5 — Check Kubernetes Events

Run:

```bash
kubectl get events -n <namespace> --sort-by=.lastTimestamp
```

Look for messages such as:

```text
Failed
Warning
BackOff
Unhealthy
FailedMount
FailedScheduling
```

---

# 9. Step 6 — Identify Root Cause

Don't write:

```text
Deployment failed.
```

That's the **symptom**, not the root cause.

Instead identify something specific.

### Example 1

```text
Symptom:
ImagePullBackOff

Root cause:
Container image does not exist.
```

### Example 2

```text
Symptom:
CrashLoopBackOff

Root cause:
Application exits because DATABASE_URL is missing.
```

### Example 3

```text
Symptom:
Pending

Root cause:
Insufficient CPU on available nodes.
```

### Example 4

```text
Symptom:
OOMKilled

Root cause:
Container exceeded its memory limit.
```

---

# 10. Step 7 — Corrective Action

Once the root cause is known, decide what action to take.

### Fix forward

For example:

```text
Wrong image
    ↓
Correct image
    ↓
Commit
    ↓
GitLab pipeline
    ↓
Deploy
```

Or:

```text
Missing ConfigMap
    ↓
Create/fix ConfigMap
    ↓
Redeploy
```

---

# 11. Step 8 — Rollback Decision

If the deployment is causing a serious production problem:

```text
Stop investigation
       ↓
Rollback
```

Command:

```bash
kubectl rollout undo deployment/<deployment-name> -n <namespace>
```

Then:

```bash
kubectl rollout status deployment/<deployment-name> -n <namespace>
```

---

# 12. Step 9 — Verify

Never finish the incident immediately after running the fix.

Check:

### Deployment

```bash
kubectl get deployment <deployment-name> -n <namespace>
```

### Pods

```bash
kubectl get pods -n <namespace>
```

### Rollout

```bash
kubectl rollout status deployment/<deployment-name> -n <namespace>
```

### Application

Check:

```text
Application health
API response
Grafana metrics
Error rate
```

### Alert

Finally:

```text
Alert = Resolved
```

---

# 13. Step 10 — Close the Incident

Record:

```text
Incident:
Failed deployment

Root cause:
Invalid container image

Impact:
Application unavailable

Corrective action:
Deployment image corrected

Rollback:
Not required

Verification:
All replicas healthy

Status:
Resolved
```

---

# 14. Decision Tree

This is the most useful part of the runbook:

```text
              Deployment Failed
                     │
                     ▼
               Check Pods
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
   ImagePullBackOff  CrashLoop   Pending
          │          │          │
          ▼          ▼          ▼
       Image       Logs/      Scheduling
       problem     Events       issue
          │          │          │
          └──────────┼──────────┘
                     ▼
                Root Cause
                     │
             ┌───────┴────────┐
             ▼                ▼
          Fix Forward      Rollback
             │                │
             └───────┬────────┘
                     ▼
                  Verify
                     │
                     ▼
                  Resolved
```

---

# 15. Quick Reference Card

During an actual incident, you don't want to read the entire runbook.

Keep this:

```bash
# 1. Deployment
kubectl get deployment -n <namespace>

# 2. Pods
kubectl get pods -n <namespace>

# 3. Details
kubectl describe pod <pod> -n <namespace>

# 4. Logs
kubectl logs <pod> -n <namespace>

# 5. Previous logs
kubectl logs <pod> --previous -n <namespace>

# 6. Events
kubectl get events -n <namespace> --sort-by=.lastTimestamp

# 7. Rollout
kubectl rollout status deployment/<name> -n <namespace>

# 8. History
kubectl rollout history deployment/<name> -n <namespace>

# 9. Rollback
kubectl rollout undo deployment/<name> -n <namespace>
```

---

# 16. Our Operations Flow So Far

You've now learned:

```text
                GitLab
                  │
               Deploy
                  ↓
              Kubernetes
                  │
             Deployment
                  │
            ┌─────┴─────┐
            │           │
          Healthy      Failed
            │           │
            │           ▼
            │        Runbook
            │           │
            │      Investigation
            │           │
            │       Root Cause
            │           │
            │      ┌────┴────┐
            │      │         │
            │     Fix      Rollback
            │      │         │
            │      └────┬────┘
            │           ↓
            └───────► Verify
                       │
                       ↓
                    Resolved
```

### What we've covered

✅ Failed deployment
✅ Kubernetes investigation
✅ Root-cause identification
✅ Fix-forward
✅ Rollback
✅ Verification
✅ Production-style runbook

We have **not yet touched security alerts or alert → Issue/MR workflows**, as you requested.

**Next lesson:** we'll create a **real incident scenario involving `CrashLoopBackOff`** and follow this runbook from detection → logs → root cause → fix → verification.
