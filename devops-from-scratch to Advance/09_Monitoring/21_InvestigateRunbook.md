# Operations & Runbooks — Lesson 4

## GitLab CI/CD Deployment Failure — Hands-on

Now we'll make the previous runbook **practical with GitLab CI/CD**.

### Our goal

We'll create a simple pipeline:

```text
GitLab
  ↓
Build
  ↓
Deploy to Kubernetes
  ↓
Rollout check
  ↓
❌ Failure
  ↓
Runbook investigation
```

We will **not cover security scans or GitLab Issues/MRs yet**.

---

# Step 1 — Create the GitLab project structure

Your repository can initially contain:

```text
my-k8s-app/
│
├── .gitlab-ci.yml
│
└── deployment.yaml
```

For now, we only need these two files.

---

# Step 2 — Create the Kubernetes Deployment

Create:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: demo-app
  namespace: default

spec:
  replicas: 2

  selector:
    matchLabels:
      app: demo-app

  template:
    metadata:
      labels:
        app: demo-app

    spec:
      containers:
        - name: app
          image: nginx:1.27
          ports:
            - containerPort: 80
```

This is our **working deployment**.

Apply it manually once to make sure your Kubernetes cluster is ready:

```bash
kubectl apply -f deployment.yaml
```

Check:

```bash
kubectl get pods
```

You should get:

```text
demo-app-xxxxx   1/1   Running
demo-app-xxxxx   1/1   Running
```

Then:

```bash
kubectl rollout status deployment/demo-app
```

Expected:

```text
deployment "demo-app" successfully rolled out
```

---

# Step 3 — Create the GitLab pipeline

Create:

```yaml
stages:
  - deploy

deploy:
  stage: deploy

  script:
    - kubectl apply -f deployment.yaml
    - kubectl rollout status deployment/demo-app --timeout=120s
```

The important command is:

```bash
kubectl rollout status deployment/demo-app --timeout=120s
```

This makes GitLab wait for Kubernetes.

---

# Step 4 — Why `kubectl apply` isn't enough

Suppose we only had:

```bash
kubectl apply -f deployment.yaml
```

Kubernetes might respond:

```text
deployment.apps/demo-app configured
```

GitLab could therefore report:

```text
✅ Job passed
```

But the Pods might subsequently fail.

That's why we add:

```bash
kubectl rollout status
```

Now GitLab can detect that the deployment did not actually become healthy.

---

# Step 5 — Intentionally break the deployment

Now we're going to simulate a real incident.

Change:

```yaml
image: nginx:1.27
```

to:

```yaml
image: nginx:does-not-exist
```

Commit and push.

GitLab will run:

```text
Deploy
  ↓
kubectl apply
  ↓
Deployment accepted
  ↓
Kubernetes creates Pods
  ↓
ImagePullBackOff
  ↓
rollout status waits
  ↓
Timeout
  ↓
❌ GitLab job fails
```

---

# Step 6 — Start the Runbook

The GitLab job has failed.

Don't immediately change the YAML.

Pretend you're the DevOps engineer receiving the incident.

### Question 1

What failed?

Go to:

**GitLab → CI/CD → Pipelines → Failed Pipeline → deploy**

Look at the job log.

You may see something similar to:

```text
error: deployment "demo-app" exceeded its progress deadline
```

Now we know:

> Kubernetes deployment did not successfully roll out.

---

# Step 7 — Check Kubernetes

Run:

```bash
kubectl get deployment demo-app
```

You might see:

```text
NAME        READY   UP-TO-DATE   AVAILABLE
demo-app    0/2     2            0
```

Then:

```bash
kubectl get pods
```

You should see:

```text
demo-app-xxxxx   0/1   ImagePullBackOff
demo-app-xxxxx   0/1   ImagePullBackOff
```

---

# Step 8 — Investigate

Take one Pod:

```bash
kubectl describe pod <pod-name>
```

Look at:

```text
Events:
```

You'll find something similar to:

```text
Failed to pull image "nginx:does-not-exist"
```

Now our investigation is:

```text
GitLab deploy failed
       ↓
Kubernetes rollout failed
       ↓
Pods = ImagePullBackOff
       ↓
Pod Events
       ↓
Image cannot be pulled
```

### Root cause

```text
Invalid container image
nginx:does-not-exist
```

---

# Step 9 — Corrective Action

We have two options.

### Option A — Fix the deployment

Change:

```yaml
image: nginx:does-not-exist
```

back to:

```yaml
image: nginx:1.27
```

Commit and push.

GitLab runs again:

```text
Deploy
 ↓
kubectl apply
 ↓
rollout status
 ↓
Successful
```

### Option B — Rollback

If this were a production incident and the previous version was known to be healthy:

```bash
kubectl rollout undo deployment/demo-app
```

Then:

```bash
kubectl rollout status deployment/demo-app
```

---

# Step 10 — Verify

After the fix:

```bash
kubectl get pods
```

Expected:

```text
demo-app-xxxxx   1/1   Running
demo-app-xxxxx   1/1   Running
```

Then:

```bash
kubectl get deployment demo-app
```

Expected:

```text
READY   UP-TO-DATE   AVAILABLE
2/2     2            2
```

And:

```bash
kubectl rollout status deployment/demo-app
```

Expected:

```text
deployment "demo-app" successfully rolled out
```

---

# Final Runbook

We've now converted the theory into a real procedure:

```text
             GitLab Deployment
                    ↓
                 FAILED
                    ↓
          Check GitLab Job Log
                    ↓
          Check Kubernetes
                    ↓
             get deployment
                    ↓
                get pods
                    ↓
             describe pod
                    ↓
             Check Events
                    ↓
              Find Root Cause
                    ↓
            ┌───────┴───────┐
            ↓               ↓
          Fix             Rollback
            ↓               ↓
            └───────┬───────┘
                    ↓
               Verify Pods
                    ↓
             Verify Rollout
                    ↓
                 RESOLVED
```

### Commands to remember

```bash
kubectl get deployment
kubectl get pods
kubectl describe pod <pod>
kubectl get events --sort-by=.lastTimestamp
kubectl logs <pod>
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>
```

**Next lesson:** we'll take this one step further and create a **proper production-style deployment runbook document** with sections such as **Symptoms → Impact → Prerequisites → Investigation → Resolution → Rollback → Verification → Escalation**. That becomes something you could actually hand to another DevOps engineer.
