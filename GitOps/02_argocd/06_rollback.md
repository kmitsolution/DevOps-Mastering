Yes. Rollback is an important Argo CD topic, but there is one **very important distinction**:

> **Argo CD rollback is primarily about going back to a previous application revision that Argo CD has recorded.**

And if you are using **automated sync**, there are some considerations around rollback because Argo CD can immediately sync back to the Git state.

# Lesson 5 — Argo CD Rollback

Let's use your `my-app` application.

---

## 1. First check application history

Run:

```bash
argocd app history my-app
```

You might see:

```text
ID    DATE                         REVISION
0     2026-08-26 10:00:00          abc1234
1     2026-08-26 11:00:00          def5678
2     2026-08-26 12:00:00          ghi9012
```

The important column is:

```text
REVISION
```

These are Git commit IDs/revisions that Argo CD has deployed.

---

# 2. See the current application

```bash
argocd app get my-app
```

For example:

```text
Sync Status:    Synced
Health Status:  Healthy
Revision:       ghi9012
```

Suppose `ghi9012` is the bad deployment.

You want to go back to:

```text
def5678
```

---

# 3. Rollback using the CLI

First check the available history:

```bash
argocd app history my-app
```

Then rollback to the desired history ID:

```bash
argocd app rollback my-app 1
```

For example:

```text
argocd app rollback my-app 1
```

Argo CD will deploy the revision represented by history ID `1`.

Then check:

```bash
argocd app get my-app
```

---

# 4. Important: Automatic Sync changes the rollback behavior

Suppose you have:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

and your Git repository currently says:

```text
Version 3
```

You rollback Argo CD to:

```text
Version 2
```

But Git still says:

```text
Version 3
```

You now have:

```text
Git              Kubernetes

Version 3   !=    Version 2
```

With automated sync/self-healing enabled, Argo CD may synchronize back to Version 3.

So for a rollback exercise, it's useful to temporarily disable automated sync:

```bash
argocd app set my-app --sync-policy none
```

Then perform the rollback.

---

# 5. A better GitOps rollback approach

In a **GitOps environment**, the preferred rollback is often:

```text
Git
 |
 | revert bad commit
 v
Argo CD
 |
 | automatic sync
 v
Kubernetes
```

For example:

```text
Commit A
   ↓
Version 1
   ↓
Commit B
   ↓
Version 2
   ↓
Commit C
   ↓
Version 3  ← BAD
```

Instead of manually rolling back Argo CD, revert Commit C:

```bash
git revert <commit-C>
git push origin main
```

Now Git becomes:

```text
Version 2
```

and Argo CD automatically deploys Version 2.

This is usually the cleaner GitOps model because **Git remains the source of truth**.

---

# 6. Practical rollback exercise

Let's make this very simple with NGINX.

### Version 1

Git:

```yaml
replicas: 2
```

Commit:

```bash
git add .
git commit -m "Deploy nginx with 2 replicas"
git push origin main
```

Argo CD:

```text
Synced
```

---

### Version 2

Change:

```yaml
replicas: 3
```

Commit:

```bash
git add .
git commit -m "Scale nginx to 3 replicas"
git push origin main
```

Argo CD automatically syncs.

Check:

```bash
kubectl get deployment nginx
```

You should see:

```text
3/3
```

---

### Version 3 — Bad change

Change:

```yaml
replicas: 10
```

Push:

```bash
git add .
git commit -m "Scale nginx to 10 replicas"
git push origin main
```

Argo CD syncs:

```text
10 replicas
```

Now imagine this is a bad deployment.

---

# 7. See the history

```bash
argocd app history my-app
```

You might see:

```text
ID    REVISION
0     abc111
1     def222
2     ghi333
```

Suppose:

```text
abc111 = Version 1
def222 = Version 2
ghi333 = Version 3
```

You want:

```text
def222
```

---

# 8. Rollback

First disable automated sync if necessary:

```bash
argocd app set my-app --sync-policy none
```

Then:

```bash
argocd app rollback my-app 1
```

Check:

```bash
kubectl get deployment nginx
```

You should see:

```text
3/3
```

instead of:

```text
10/10
```

---

# 9. Check the rollback

```bash
argocd app get my-app
```

and:

```bash
argocd app history my-app
```

You should be able to see the application history and the revision associated with the rollback.

---

# 10. Rollback from the Argo CD UI

You can also do this from the UI.

Open:

```text
my-app
```

Then look for:

```text
HISTORY AND ROLLBACK
```

Select the previous revision.

Then choose:

```text
ROLLBACK
```

Argo CD will deploy that historical revision.

---

# 11. `rollback` vs `sync`

These are different.

### Sync

```bash
argocd app sync my-app
```

means:

> Deploy the desired state currently defined by the Application/Git.

### Rollback

```bash
argocd app rollback my-app <history-id>
```

means:

> Go back to a previously recorded application revision.

Think:

```text
SYNC

Git
 ↓
Current desired state
 ↓
Kubernetes
```

versus:

```text
ROLLBACK

Argo CD History
 ↓
Previous revision
 ↓
Kubernetes
```

---

# 12. Production GitOps recommendation

For your course, I would teach both approaches.

### Operational rollback

Useful when you need an immediate recovery:

```bash
argocd app rollback my-app <history-id>
```

### Git rollback

Preferred when Git is the source of truth:

```bash
git revert <bad-commit>
git push
```

Then:

```text
Git
 ↓
Argo CD detects change
 ↓
Automatic Sync
 ↓
Previous good version
```

This second approach gives you an **auditable rollback** because the Git history clearly shows who reverted the deployment and why.

---

## One important limitation with automated sync

If you have:

```yaml
syncPolicy:
  automated:
    selfHeal: true
```

don't think of an Argo CD rollback as permanently changing your desired state.

For example:

```text
Git says:       v3
Rollback to:    v2
Self-heal sees: v2 != v3
                    ↓
                  v3 again
```

That's why for a **permanent rollback**, change Git:

```text
v3
 ↓
git revert
 ↓
v2
 ↓
Argo CD
 ↓
Kubernetes v2
```

### The key rule to remember

> **Argo CD rollback changes what is deployed; Git revert changes what is desired.**

For a production GitOps workflow, **Git revert is usually the preferred permanent rollback mechanism**, while Argo CD rollback is useful for fast operational recovery/testing.
