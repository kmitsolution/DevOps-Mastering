Exactly. **Pull-based deployment from a GitLab repository** is the important GitOps concept you want to learn with Argo CD.

Let's use your existing repository:

[GitLab `argocd-demo` repository](https://gitlab.com/raja-group3312134/argocd-demo?utm_source=chatgpt.com)

## Pull-based deployment — simple example

Your architecture is:

```text
Developer
    |
    | git push
    v
GitLab Repository
    |
    | deploy.yaml
    |
    v
   Argo CD
    |
    | pulls/checks Git
    v
Kubernetes Cluster
```

The important point is:

> **GitLab does not push the deployment to Kubernetes. Argo CD pulls the desired state from GitLab.**

---

# 1. GitLab repository

Suppose your repository contains:

```text
argocd-demo/
│
└── deploy.yaml
```

`deploy.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 2
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
          ports:
            - containerPort: 80
```

---

# 2. Create Argo CD Application

Your Argo CD Application points to GitLab:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application

metadata:
  name: nginx-app
  namespace: argocd

spec:
  project: default

  source:
    repoURL: https://gitlab.com/raja-group3312134/argocd-demo.git
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
kubectl apply -f application.yaml
```

---

# 3. What Argo CD does

Argo CD now knows:

```text
Repository:
https://gitlab.com/raja-group3312134/argocd-demo.git

Branch:
main

Path:
.
```

It periodically checks GitLab.

Conceptually:

```text
        GitLab
          |
          | "What is the desired state?"
          |
          v
       Argo CD
          |
          | compare
          |
          +----------------+
          |                |
          v                v
      Desired           Actual
       state             state
       Git             Kubernetes
          |                |
          +-------+--------+
                  |
              Difference?
                  |
                 YES
                  |
                  v
                Sync
                  |
                  v
              Kubernetes
```

---

# 4. Initial deployment

After creating the Application:

```bash
argocd app get nginx-app
```

You might initially see:

```text
Sync Status: OutOfSync
Health Status: Missing
```

Because Git contains:

```yaml
replicas: 2
```

but Kubernetes doesn't have the Deployment yet.

Because we enabled:

```yaml
automated:
  prune: true
  selfHeal: true
```

Argo CD automatically syncs it.

Check:

```bash
kubectl get deployment
```

You should see:

```text
NAME    READY   UP-TO-DATE   AVAILABLE
nginx   2/2     2            2
```

---

# 5. Now the real GitOps test

Change your GitLab `deploy.yaml`.

For example:

```yaml
replicas: 3
```

Previously:

```yaml
replicas: 2
```

Commit and push:

```bash
git add deploy.yaml
git commit -m "Scale nginx to 3 replicas"
git push origin main
```

**You do NOT run:**

```bash
kubectl apply
```

and you do NOT run:

```bash
argocd app sync nginx-app
```

because we enabled automated sync.

Argo CD detects:

```text
GitLab
replicas: 3

        ↓

Argo CD

        ↓

Kubernetes
replicas: 2
```

It sees:

```text
OutOfSync
```

and automatically synchronizes.

Eventually:

```bash
kubectl get deployment nginx
```

shows:

```text
READY
3/3
```

---

# 6. This is Pull-Based Deployment

Notice who initiates the deployment.

### Push model

Traditional CI/CD:

```text
GitLab CI
   |
   | kubectl apply
   v
Kubernetes
```

GitLab is **pushing** to Kubernetes.

### Pull model

Argo CD:

```text
GitLab
   ^
   |
   | Argo CD pulls desired state
   |
Argo CD
   |
   v
Kubernetes
```

Argo CD is responsible for deployment.

---

# 7. Why this is better for GitOps

Suppose GitLab CI finishes successfully.

With traditional deployment:

```text
GitLab CI
   |
   | kubectl
   v
Kubernetes
```

The CI runner needs Kubernetes credentials.

You have to give GitLab access to:

```text
Kubernetes API
```

With Argo CD:

```text
GitLab
   |
   | Repository
   |
   v
Argo CD
   |
   | Kubernetes credentials
   v
Kubernetes
```

The GitLab Runner doesn't need direct Kubernetes access.

That's a major advantage.

---

# 8. What does GitLab CI do then?

This is where GitLab CI/CD fits nicely.

GitLab CI can do:

```text
GitLab
   |
   v
GitLab CI
   |
   +-- Unit Tests
   +-- Security Scan
   +-- Build Docker Image
   +-- Push Image
   |
   v
Update deployment.yaml
   |
   v
GitLab Repository
   |
   v
Argo CD
   |
   v
Kubernetes
```

So:

| Component       | Responsibility      |
| --------------- | ------------------- |
| GitLab          | Source control      |
| GitLab CI       | Build/Test/Security |
| GitLab Registry | Container images    |
| Argo CD         | Deployment          |
| Kubernetes      | Runtime             |

---

# 9. Very simple `.gitlab-ci.yml`

For your current learning exercise, we can keep CI extremely simple:

```yaml
stages:
  - validate

validate:
  stage: validate
  image:
    name: bitnami/kubectl:latest
    entrypoint: [""]
  script:
    - kubectl apply --dry-run=client -f deploy.yaml
```

Now the workflow is:

```text
git push
   |
   v
GitLab CI
   |
   | Validate deploy.yaml
   |
   v
GitLab Repository
   |
   | Argo CD pulls
   v
Argo CD
   |
   | Automated Sync
   v
Kubernetes
```

**CI doesn't deploy. Argo CD deploys.**

That is the cleanest way to demonstrate **pull-based deployment from GitLab repositories**.

---

## One thing to remember

### GitLab doesn't need to know where Kubernetes is.

Argo CD needs to know:

```text
GitLab Repository
        ↓
Argo CD
        ↓
Kubernetes
```

And because Argo CD is running **inside your Kubernetes cluster**, it can communicate with the Kubernetes API directly.

This is why the Application destination:

```yaml
destination:
  server: https://kubernetes.default.svc
```

works.

**Next logical lesson:** make the GitLab repository **private** and configure Argo CD with a GitLab username/token so you can understand how Argo CD pulls from a private GitLab repository.
