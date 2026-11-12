# Operations & Runbooks — Lesson 2

## Failed Deployment → Rollback

Now that we know how to **investigate a failed deployment**, the next practical step is:

> **What do we do when the new version is broken and we need to quickly restore the previous working version?**

This is where **rollback** comes in.

---

## 1. What is a Rollback?

Suppose production is running:

```text
v1
 ↓
Working ✅
```

We deploy:

```text
v2
 ↓
Deployment
 ↓
Pods failing ❌
```

Instead of spending 30 minutes fixing v2 while users are affected, we can return to v1:

```text
v2 ❌
 ↓
ROLLBACK
 ↓
v1 ✅
```

The basic operational flow is:

```text
Deployment
    ↓
Problem detected
    ↓
Investigate
    ↓
New version is confirmed as cause
    ↓
Rollback
    ↓
Verify
```

---

# 2. Kubernetes Deployment History

Kubernetes keeps revisions of a Deployment.

Check the history:

```bash
kubectl rollout history deployment/<deployment-name>
```

For our example:

```bash
kubectl rollout history deployment/broken-app
```

You may see:

```text
deployment.apps/broken-app
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

Think of these as:

```text
Revision 1 → v1
Revision 2 → v2
```

---

# 3. See the Current Deployment

Run:

```bash
kubectl get deployment broken-app
```

Then:

```bash
kubectl describe deployment broken-app
```

You can see information about the current ReplicaSet/revision.

---

# 4. Create a Real Revision

Let's make our example slightly more realistic.

Change the image:

```yaml
image: nginx:1.27
```

Apply:

```bash
kubectl apply -f failed-deployment.yaml
```

Check:

```bash
kubectl rollout status deployment/broken-app
```

Then:

```bash
kubectl rollout history deployment/broken-app
```

We now have another deployment revision.

---

# 5. Simulate a Bad Deployment

Now deliberately change the image to something invalid:

```yaml
image: nginx:does-not-exist
```

Apply:

```bash
kubectl apply -f failed-deployment.yaml
```

Check:

```bash
kubectl get pods
```

You should see something like:

```text
broken-app-xxxxx   0/1   ImagePullBackOff
```

Now we have our incident:

```text
New deployment
      ↓
Pods fail
      ↓
ImagePullBackOff
      ↓
New version is bad
```

---

# 6. Check Rollout Status

Run:

```bash
kubectl rollout status deployment/broken-app
```

It may remain waiting because the new Pods aren't becoming ready.

You can also check:

```bash
kubectl get deployment broken-app
```

---

# 7. Rollback

Now comes the important command:

```bash
kubectl rollout undo deployment/broken-app
```

Kubernetes will return the Deployment to the previous revision.

You should see:

```text
deployment.apps/broken-app rolled back
```

---

# 8. Watch the Pods

Run:

```bash
kubectl get pods -w
```

You should see Kubernetes terminating the bad Pods and bringing back the previous version.

Eventually:

```text
READY   STATUS
1/1     Running
1/1     Running
```

Press:

```text
Ctrl+C
```

---

# 9. Verify Rollout

Run:

```bash
kubectl rollout status deployment/broken-app
```

Expected:

```text
deployment "broken-app" successfully rolled out
```

Then:

```bash
kubectl get deployment broken-app
```

We want:

```text
READY   UP-TO-DATE   AVAILABLE
2/2     2            2
```

---

# 10. Check History Again

Run:

```bash
kubectl rollout history deployment/broken-app
```

You can now see the revisions.

```text
Revision 1 → Working version
Revision 2 → Bad version
Revision 3 → Rollback result
```

The exact revision numbering can vary depending on what changes you made.

---

# 11. Rollback Runbook

We can now add this to our operational runbook.

## RUNBOOK — Failed Deployment

### Detect

```bash
kubectl get deployments
kubectl get pods
```

### Investigate

```bash
kubectl describe pod <pod>
kubectl get events --sort-by=.lastTimestamp
kubectl logs <pod>
```

### Decide

If the new deployment is confirmed to be the cause:

```text
Rollback required
```

### Rollback

```bash
kubectl rollout undo deployment/<deployment-name>
```

### Monitor

```bash
kubectl get pods -w
```

### Verify

```bash
kubectl rollout status deployment/<deployment-name>
```

and:

```bash
kubectl get deployment <deployment-name>
```

---

# 12. The Complete Incident Flow

This is what you should remember:

```text
                Deployment
                    ↓
              New version v2
                    ↓
              Pods unhealthy
                    ↓
                 Alert
                    ↓
              Investigation
                    ↓
           Root cause = v2
                    ↓
                Rollback
                    ↓
                 v1
                    ↓
              Pods healthy
                    ↓
              Verify system
                    ↓
               RESOLVED
```

### Key commands to remember

```bash
kubectl get pods
kubectl describe pod <pod>
kubectl get events --sort-by=.lastTimestamp
kubectl rollout history deployment/<deployment>
kubectl rollout status deployment/<deployment>
kubectl rollout undo deployment/<deployment>
```

**Next in this runbook topic:** we'll connect this workflow to **GitLab CI/CD** — a GitLab deployment job fails, the DevOps engineer investigates Kubernetes, and the corrective action/rollback is recorded against the GitLab pipeline.
