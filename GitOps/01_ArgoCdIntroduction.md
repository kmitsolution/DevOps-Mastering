# Lesson 1 — Introduction to GitOps and Argo CD

We’ll do this **as a hands-on tutorial**, one lesson at a time. Don't worry about GitLab integration yet; we'll introduce it after you understand the Argo CD basics.

## 1. What are we going to build?

Our final setup will be:

```text
                    GitLab
                      |
                      | Git Push
                      v
                 +----------+
                 |  Argo CD |
                 +----+-----+
                      |
                      | Sync
                      v
              +---------------+
              |  Kubernetes   |
              |    Cluster    |
              +---------------+
                      |
                  Application
                     Pods
```

For Lesson 1, we'll focus on understanding this architecture.

---

## 2. First: What problem does Argo CD solve?

Suppose you have this Kubernetes Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
```

You could deploy it manually:

```bash
kubectl apply -f deployment.yaml
```

This works.

But imagine that your company has **100 applications** and **10 Kubernetes clusters**.

You don't want developers manually running:

```bash
kubectl apply
```

against production.

You want Git to define what production **should look like**.

That's where GitOps comes in.

---

# 3. What is GitOps?

GitOps means:

> **Git is the source of truth for your infrastructure and application configuration.**

For example, GitLab contains:

```text
deployment.yaml
service.yaml
ingress.yaml
configmap.yaml
```

These files describe your desired Kubernetes environment.

```text
GitLab
   |
   | Desired State
   |
   v
Argo CD
   |
   | Reconciliation
   |
   v
Kubernetes
```

---

# 4. Desired State vs Actual State

This is the most important concept in Argo CD.

Suppose GitLab contains:

```yaml
replicas: 3
```

That means:

```text
Desired State = 3 replicas
```

But Kubernetes currently has:

```text
2 replicas
```

Therefore:

```text
Desired State       Actual State

    3                    2
    |                    |
    +--------+-----------+
             |
             v
         Different
```

Argo CD detects this difference.

It calls this:

```text
OUT OF SYNC
```

---

# 5. What happens when they match?

Suppose:

```text
GitLab:

replicas: 3
```

and:

```text
Kubernetes:

replicas: 3
```

Then:

```text
Desired State = Actual State
```

Argo CD reports:

```text
SYNCED
```

So you can remember:

```text
Git                  Kubernetes
 |                        |
 |   desired state        | actual state
 |                        |
 +----------+-------------+
            |
         Argo CD
            |
            v
       Compare them
```

---

# 6. What is Reconciliation?

Suppose:

```text
GitLab
replicas: 3
```

but someone manually runs:

```bash
kubectl scale deployment nginx --replicas=1
```

Now:

```text
GitLab             Kubernetes

3 replicas         1 replica
    |                  |
    +--------+---------+
             |
             v
        Argo CD detects
          the difference
```

Argo CD can synchronize Kubernetes back to:

```text
3 replicas
```

This process is called:

> **Reconciliation**

Think of Argo CD as continuously asking:

```text
"What does Git say?"

"What is actually running?"

"Are they the same?"
```

---

# 7. Traditional Deployment vs GitOps

### Traditional CI/CD

```text
Developer
    |
    v
 GitLab
    |
    v
GitLab CI
    |
    | kubectl apply
    v
Kubernetes
```

The CI pipeline directly changes Kubernetes.

---

### GitOps

```text
Developer
    |
    v
 GitLab
    |
    | Desired configuration
    v
 Argo CD
    |
    | Reconciliation
    v
Kubernetes
```

Argo CD is responsible for deployment synchronization.

This is why Argo CD is often called a:

> **GitOps Continuous Delivery tool for Kubernetes.**

---

# 8. Push vs Pull

This is an important interview question.

### Push model

```text
GitLab CI
    |
    | PUSH
    v
Kubernetes
```

The pipeline pushes changes into Kubernetes.

### Pull model

```text
GitLab
    ^
    |
    | Pull configuration
    |
Argo CD
    |
    v
Kubernetes
```

Argo CD continuously monitors the Git repository and reconciles the cluster.

So:

**GitLab CI → Kubernetes** = generally a push model.

**Argo CD → Git + Kubernetes reconciliation** = GitOps/pull-oriented model.

---

# 9. Why GitLab?

We'll use GitLab as our Git repository.

For example:

```text
GitLab Repository

argocd-demo/
│
├── deployment.yaml
├── service.yaml
└── ingress.yaml
```

GitLab stores the desired state.

Argo CD reads it.

Kubernetes runs it.

So the responsibility becomes:

| Component          | Responsibility                 |
| ------------------ | ------------------------------ |
| GitLab             | Store desired configuration    |
| GitLab CI          | Build/test/package application |
| Container Registry | Store Docker images            |
| Argo CD            | Deploy/reconcile               |
| Kubernetes         | Run application                |

This separation becomes very important when we build the complete project later.

---

# 10. Your First Lab Check

Since you already have a Kubernetes lab, let's verify it before installing Argo CD.

Run:

```bash
kubectl get nodes
```

You should get something similar to:

```text
NAME       STATUS   ROLES           AGE
master     Ready    control-plane   20d
worker1    Ready    <none>          20d
worker2    Ready    <none>          20d
```

The important thing is:

```text
STATUS
------
Ready
```

---

## 11. Check Kubernetes Pods

Run:

```bash
kubectl get pods -A
```

You should see Kubernetes system pods.

They should generally be:

```text
Running
```

or other appropriate completed states for jobs.

---

# 12. Create the Argo CD Namespace

We'll eventually install Argo CD into:

```text
argocd
```

Create it:

```bash
kubectl create namespace argocd
```

Verify:

```bash
kubectl get namespaces
```

You should see:

```text
argocd
default
kube-system
...
```

Then:

```bash
kubectl get namespace argocd
```

Expected:

```text
NAME      STATUS   AGE
argocd    Active   ...
```

---

# 13. Important: Don't Install Argo CD Yet

For **Lesson 1**, stop here.

I want you to understand the following flow first:

```text
             GitLab
                |
                | Desired State
                v
            +-------+
            | Argo  |
            |  CD   |
            +---+---+
                |
             Compare
                |
       +--------+--------+
       |                 |
       v                 v
   Desired State      Live State
       |                 |
       |                 |
       +-------+---------+
               |
               v
          Reconciliation
               |
               v
          Kubernetes
```

### Remember these four words:

**Git → Desired State → Argo CD → Reconciliation**

---

## Lesson 1 Exercise

Run these commands:

```bash
kubectl get nodes
```

```bash
kubectl get pods -A
```

```bash
kubectl create namespace argocd
```

```bash
kubectl get namespace argocd
```

If they work successfully, **Lesson 1 is complete**.

Then we'll move to **Lesson 2: Installing Argo CD on your Kubernetes cluster**, where we'll install it step-by-step and inspect every component instead of treating the installation command as a black box.
