# Operations & Runbooks — Lesson 3

## Failed Deployment → GitLab CI/CD

Now we'll connect the runbook to **GitLab**, but we'll keep the scope limited to **failed deployments**.

The scenario:

```text
Developer
   ↓
Git push
   ↓
GitLab CI/CD
   ↓
Deploy to Kubernetes
   ↓
Deployment fails
   ↓
DevOps investigates
   ↓
Fix / Rollback
   ↓
Verify
```

---

## 1. Where does the incident start?

Suppose our GitLab pipeline has:

```yaml
deploy:
  stage: deploy
  script:
    - kubectl apply -f deployment.yaml
```

GitLab runs the deployment.

The job might show:

```text
deploy
  ↓
kubectl apply
  ↓
deployment created
  ↓
❌ Pods not becoming Ready
```

Important distinction:

> **GitLab job success does not always mean application deployment success.**

For example:

```bash
kubectl apply -f deployment.yaml
```

can return successfully because Kubernetes accepted the YAML.

But the application might subsequently enter:

```text
CrashLoopBackOff
ImagePullBackOff
Pending
```

So a good deployment pipeline should also **wait for rollout success**.

---

# 2. Improve the GitLab deployment job

Instead of:

```yaml
deploy:
  stage: deploy
  script:
    - kubectl apply -f deployment.yaml
```

use:

```yaml
deploy:
  stage: deploy

  script:
    - kubectl apply -f deployment.yaml
    - kubectl rollout status deployment/broken-app --timeout=120s
```

Now GitLab waits for Kubernetes to report rollout status.

If the rollout succeeds:

```text
deployment successfully rolled out
```

GitLab job:

```text
✅ PASSED
```

If the Pods don't become ready:

```text
❌ Job FAILED
```

This is much better operationally.

---

# 3. Our Incident Scenario

Imagine the developer changes:

```yaml
image: nginx:1.27
```

to:

```yaml
image: nginx:does-not-exist
```

GitLab executes:

```text
kubectl apply
      ↓
Kubernetes accepts deployment
      ↓
Pod created
      ↓
ImagePullBackOff
      ↓
rollout status waits
      ↓
Timeout
      ↓
GitLab job FAILED
```

Now the runbook starts.

---

# 4. Failed Deployment Runbook

### Step 1 — Check GitLab

Look at:

**GitLab → CI/CD → Pipelines**

Find the failed pipeline.

Then:

**Pipeline → deploy job**

Check the error.

For example:

```text
error: deployment "broken-app" exceeded its progress deadline
```

---

# 5. Step 2 — Check Kubernetes

Run:

```bash
kubectl get deployment
```

Then:

```bash
kubectl get pods
```

Suppose:

```text
broken-app-xxxxx   0/1   ImagePullBackOff
```

Now investigate.

---

# 6. Step 3 — Describe the Pod

```bash
kubectl describe pod <pod-name>
```

Look at:

```text
Events:
```

Suppose:

```text
Failed to pull image "nginx:does-not-exist"
```

Root cause:

```text
❌ Invalid container image
```

---

# 7. Step 4 — Decide: Fix or Rollback?

Now the engineer decides:

### If the fix is small and safe

Fix the deployment:

```text
Wrong image
   ↓
Correct image
   ↓
Deploy again
```

### If production is badly affected

Rollback:

```bash
kubectl rollout undo deployment/broken-app
```

So the runbook decision becomes:

```text
                Deployment failed
                       ↓
                 Investigate
                       ↓
                 Root cause?
                  /       \
                Fix       Rollback
                 ↓           ↓
              Deploy      Previous
                ↓          version
                └─────┬──────┘
                      ↓
                   Verify
```

---

# 8. Step 5 — Verify

After fixing or rolling back:

```bash
kubectl rollout status deployment/broken-app
```

Then:

```bash
kubectl get pods
```

We want:

```text
1/1   Running
1/1   Running
```

Then verify the application itself.

---

# 9. The GitLab + Kubernetes Runbook

This is the version I recommend keeping in your course notes.

## RUNBOOK — Failed Deployment

### Detection

**Source:**

```text
GitLab Pipeline
Grafana
Prometheus / Alertmanager
```

### Investigation

```bash
kubectl get deployment
kubectl get pods
kubectl describe pod <pod-name>
kubectl get events --sort-by=.lastTimestamp
kubectl logs <pod-name>
```

### Root Cause

Determine whether the failure is caused by:

```text
Wrong image
Configuration
Secret
ConfigMap
Resource limits
Application crash
Readiness probe
Liveness probe
Scheduling
Networking
```

### Corrective Action

Either:

```text
Fix → deploy again
```

or:

```text
Rollback → previous working version
```

### Verification

```bash
kubectl rollout status deployment/<name>
kubectl get pods
```

Then verify application health.

### Close

Record:

```text
What failed?
Why did it fail?
What fixed it?
Was rollback required?
How can we prevent it?
```

---

# 10. Why this is important in DevOps

This connects three things we've already learned:

```text
GitLab
   │
   │ Deployment
   ↓
Kubernetes
   │
   │ Metrics
   ↓
Prometheus
   │
   │ Alert
   ↓
Alertmanager
   │
   ↓
DevOps Engineer
   │
   ↓
Runbook
   │
   ├── Investigate
   ├── Fix
   └── Rollback
```

That's the complete operational picture.

---

## Next lesson

We'll do **one practical exercise**:

> **Create a GitLab CI/CD deployment that intentionally fails, then use this runbook to investigate and recover it.**

We'll build it incrementally, starting with a very small `.gitlab-ci.yml`.
