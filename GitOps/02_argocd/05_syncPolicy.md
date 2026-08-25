Absolutely. Since your Argo CD installation and CLI are working, let's make **Sync Policy** our next lesson.

# Lesson 4 — Argo CD Sync Policy

The easiest way to understand Sync Policy is:

> **Sync Policy tells Argo CD when and how it should apply the Kubernetes manifests from Git to the cluster.**

The basic flow is:

```text
Git Repository
      |
      | manifests
      v
   Argo CD
      |
      | Sync
      v
 Kubernetes Cluster
```

There are two main approaches:

1. **Manual Sync**
2. **Automated Sync (GitOps)**

---

# 1. Manual Sync

First, let's use manual synchronization because it makes the GitOps process easy to understand.

Your Application can look like this:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application

metadata:
  name: nginx-app
  namespace: argocd

spec:
  project: default

  source:
    repoURL: https://github.com/onlineTrainingguy/argocd.git
    targetRevision: main
    path: .

  destination:
    server: https://kubernetes.default.svc
    namespace: default
```

Notice there is **no `syncPolicy`**.

That means Argo CD uses manual synchronization.

Apply:

```bash
kubectl apply -f nginx-app.yaml
```

Then:

```bash
argocd app get nginx-app
```

You might see:

```text
Name:               nginx-app
Project:            default
Server:             https://kubernetes.default.svc
Namespace:          default
Status:             OutOfSync
Health:             Missing
```

This is expected.

Argo CD knows what should exist, but hasn't applied it yet.

---

# 2. Manual Sync using CLI

Now run:

```bash
argocd app sync nginx-app
```

You should see Argo CD applying your resources.

For example:

```text
TIMESTAMP                  GROUP        KIND     NAMESPACE   NAME
...                        apps         Deployment default    nginx
...                        core         Service   default    nginx
```

Then check:

```bash
argocd app get nginx-app
```

You want:

```text
Sync Status:    Synced
Health Status:  Healthy
```

And Kubernetes:

```bash
kubectl get pods -n default
```

You should see your NGINX Pod.

---

# 3. Manual Sync from the UI

You can also do exactly the same thing from the Argo CD UI.

Open your application:

```text
nginx-app
```

You'll see something like:

```text
OUT OF SYNC
```

Click:

```text
SYNC
```

Then:

```text
SYNCHRONIZE
```

Argo CD applies the manifests.

---

# 4. Now the Important Part — Automated Sync

This is where Argo CD becomes a real **GitOps tool**.

Add:

```yaml
syncPolicy:
  automated: {}
```

Your Application becomes:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application

metadata:
  name: nginx-app
  namespace: argocd

spec:
  project: default

  source:
    repoURL: https://github.com/onlineTrainingguy/argocd.git
    targetRevision: main
    path: .

  destination:
    server: https://kubernetes.default.svc
    namespace: default

  syncPolicy:
    automated: {}
```

Apply:

```bash
kubectl apply -f nginx-app.yaml
```

Now Argo CD has:

```text
Git
 |
 | change
 v
Argo CD
 |
 | automatically sync
 v
Kubernetes
```

You no longer have to run:

```bash
argocd app sync nginx-app
```

for every Git change.

---

# 5. Test Automated Sync

Let's do a simple experiment.

Suppose your Git repository currently contains:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 2
```

Argo CD will create:

```text
nginx
replicas = 2
```

Now change Git:

```yaml
replicas: 3
```

Commit:

```bash
git add .
git commit -m "Increase nginx replicas"
git push origin main
```

Argo CD detects the new Git revision.

Then:

```text
Git
 |
 | replicas: 3
 v
Argo CD
 |
 | automatic sync
 v
Kubernetes
 |
 v
3 nginx Pods
```

Check:

```bash
kubectl get deployment nginx
```

You should eventually see:

```text
NAME    READY   UP-TO-DATE   AVAILABLE
nginx   3/3     3            3
```

---

# 6. `prune`

Now we have another important setting.

```yaml
syncPolicy:
  automated:
    prune: true
```

Example:

```yaml
syncPolicy:
  automated:
    prune: true
```

### What does prune mean?

Suppose Git contains:

```text
deployment.yaml
service.yaml
configmap.yaml
```

Argo CD creates:

```text
Deployment
Service
ConfigMap
```

Then you delete:

```text
configmap.yaml
```

from Git.

Without pruning:

```text
Git                    Kubernetes

Deployment      --->   Deployment
Service         --->   Service
ConfigMap       X      ConfigMap remains
```

With:

```yaml
prune: true
```

Argo CD removes the ConfigMap:

```text
Git                    Kubernetes

Deployment      --->   Deployment
Service         --->   Service
ConfigMap       X      ConfigMap deleted
```

This is very important for GitOps.

---

# 7. `selfHeal`

Another important option:

```yaml
selfHeal: true
```

So a common configuration is:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

This gives us:

```text
                 Git
                  |
                  v
               Argo CD
              /       \
             /         \
        Git changes   Drift detection
             |             |
             v             v
        Kubernetes <---- Self Heal
```

---

# 8. What is Self-Healing?

Suppose Git says:

```yaml
replicas: 3
```

Kubernetes currently has:

```text
replicas = 3
```

Someone manually executes:

```bash
kubectl scale deployment nginx --replicas=1
```

Now:

```text
Git:         3
Kubernetes:  1
```

That's called **drift**.

With:

```yaml
selfHeal: true
```

Argo CD notices:

```text
Desired state = 3
Actual state  = 1
```

and changes Kubernetes back to:

```text
3
```

That's one of the biggest benefits of GitOps.

---

# 9. Recommended Learning Configuration

For your lab, I'd use:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

Complete example:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application

metadata:
  name: nginx-app
  namespace: argocd

spec:
  project: default

  source:
    repoURL: https://github.com/onlineTrainingguy/argocd.git
    targetRevision: main
    path: .

  destination:
    server: https://kubernetes.default.svc
    namespace: default

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Apply:

```bash
kubectl apply -f nginx-app.yaml
```

Then:

```bash
argocd app get nginx-app
```

You should see:

```text
Sync Status: Synced
Health Status: Healthy
```

---

# 10. `allowEmpty`

There is another option:

```yaml
allowEmpty: true
```

For example:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
    allowEmpty: true
```

This allows Argo CD to sync even when the Git path contains **no resources**.

For now, **don't enable this** in your lab.

---

# 11. Sync Policy Summary

| Configuration      | Meaning                           |
| ------------------ | --------------------------------- |
| No `syncPolicy`    | Manual sync                       |
| `automated: {}`    | Automatically sync Git changes    |
| `prune: true`      | Delete resources removed from Git |
| `selfHeal: true`   | Correct manual changes/drift      |
| `allowEmpty: true` | Allow zero resources              |

The configuration you'll commonly see in GitOps environments is:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

---

# 12. CLI Commands You Should Know

Since you've installed the Argo CD CLI, these are worth practicing:

### List applications

```bash
argocd app list
```

### Application details

```bash
argocd app get nginx-app
```

### Manual sync

```bash
argocd app sync nginx-app
```

### Application history

```bash
argocd app history nginx-app
```

### Refresh application

```bash
argocd app get nginx-app --refresh
```

### Application resources

```bash
argocd app resources nginx-app
```

### Delete application

```bash
argocd app delete nginx-app
```

---

## 🎯 Practice Exercise

I'd suggest doing this experiment because it makes **Sync + Prune + Self-Heal** very clear.

Start with:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

Then:

1. Push NGINX with `replicas: 2`.
2. Verify Argo CD shows `Synced`.
3. Change Git to `replicas: 3`.
4. Push to Git.
5. Watch Argo CD automatically change Kubernetes to 3.
6. Run:

   ```bash
   kubectl scale deployment nginx --replicas=1
   ```
7. Watch Argo CD **self-heal** it back to 3.
8. Delete `service.yaml` from Git.
9. Push.
10. Watch Argo CD **prune** the Service.

That single exercise will give you a very solid understanding of **Argo CD Sync Policy** before we move to the next lesson.
