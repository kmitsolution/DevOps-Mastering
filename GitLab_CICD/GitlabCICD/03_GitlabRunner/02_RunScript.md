Yes. Since your **GitLab Runner has the tag `localrunner`**, and Kubernetes is installed on the same `kmaster` machine, we can create a simple GitLab CI/CD pipeline that:

1. Uses your `localrunner`
2. Checks Kubernetes
3. Creates a namespace
4. Deploys an NGINX application
5. Verifies the deployment
6. Displays pods/services
7. Deletes the deployment

### Important point about `root`

Because Kubernetes was installed/configured as `root`, the `gitlab-runner` user may **not** be able to execute:

```bash
kubectl get nodes
```

even though it works for root.

First, on your `kmaster`, check:

```bash
sudo -i
kubectl get nodes
```

Then check:

```bash
sudo -u gitlab-runner kubectl get nodes
```

If the second command fails with something like:

```text
The connection to the server localhost:8080 was refused
```

or:

```text
permission denied
```

we need to give the `gitlab-runner` user access to the Kubernetes kubeconfig.

## 1. Configure kubectl for `gitlab-runner`

On `kmaster`, as root:

```bash
sudo -i
```

Check the root kubeconfig:

```bash
ls -l /root/.kube/config
```

Create a kubeconfig directory for the runner:

```bash
mkdir -p /home/gitlab-runner/.kube
```

Copy the configuration:

```bash
cp /root/.kube/config /home/gitlab-runner/.kube/config
```

Change ownership:

```bash
chown -R gitlab-runner:gitlab-runner /home/gitlab-runner/.kube
```

Set permissions:

```bash
chmod 700 /home/gitlab-runner/.kube
chmod 600 /home/gitlab-runner/.kube/config
```

Now test:

```bash
sudo -u gitlab-runner kubectl get nodes
```

You should get something similar to:

```text
NAME      STATUS   ROLES           AGE   VERSION
kmaster   Ready    control-plane   ...   v1.xx.x
```

### Why are we doing this?

Your setup is currently:

```text
root
 |
 +-- /root/.kube/config
 |
 +-- kubectl
 |
 +-- Kubernetes
```

But GitLab Runner normally executes jobs as:

```text
gitlab-runner
```

Therefore we need:

```text
gitlab-runner
      |
      +-- ~/.kube/config
      |
      +-- kubectl
      |
      v
 Kubernetes API Server
```

---

# 2. Verify kubectl path

Check:

```bash
which kubectl
```

For example:

```text
/usr/bin/kubectl
```

Then:

```bash
sudo -u gitlab-runner which kubectl
```

If it returns:

```text
/usr/bin/kubectl
```

you're ready.

---

# 3. Create `.gitlab-ci.yml`

Now create this file in your GitLab repository:

```yaml
stages:
  - validate
  - deploy
  - verify
  - cleanup

variables:
  KUBE_NAMESPACE: gitlab-demo

check_kubernetes:
  stage: validate

  tags:
    - localrunner

  script:
    - echo "Checking Kubernetes cluster..."
    - kubectl version --client
    - kubectl get nodes
    - kubectl get pods -A

deploy_application:
  stage: deploy

  tags:
    - localrunner

  script:
    - echo "Creating namespace..."
    - kubectl create namespace $KUBE_NAMESPACE --dry-run=client -o yaml | kubectl apply -f -

    - echo "Deploying NGINX..."
    - kubectl create deployment nginx --image=nginx:latest -n $KUBE_NAMESPACE

    - echo "Exposing NGINX..."
    - kubectl expose deployment nginx --port=80 --target-port=80 --type=NodePort -n $KUBE_NAMESPACE

verify_application:
  stage: verify

  tags:
    - localrunner

  script:
    - echo "Checking deployment..."
    - kubectl get deployment -n $KUBE_NAMESPACE

    - echo "Checking pods..."
    - kubectl get pods -n $KUBE_NAMESPACE

    - echo "Checking service..."
    - kubectl get service -n $KUBE_NAMESPACE

    - echo "Waiting for deployment..."
    - kubectl rollout status deployment/nginx -n $KUBE_NAMESPACE --timeout=120s

cleanup:
  stage: cleanup

  tags:
    - localrunner

  script:
    - echo "Cleaning up Kubernetes resources..."
    - kubectl delete namespace $KUBE_NAMESPACE --ignore-not-found=true
```

---

# 4. Understand the pipeline

The pipeline is:

```text
                GitLab
                   |
                   v
             CI Pipeline
                   |
        +----------+----------+
        |                     |
        v                     |
      Runner                  |
   localrunner                |
        |                     |
        v                     |
     kmaster                  |
        |                     |
        v                     |
   Kubernetes                 |
        |                     |
   +----+----+----+           |
   |         |    |           |
   v         v    v           |
Validate   Deploy Verify   Cleanup
```

Because all jobs have:

```yaml
tags:
  - localrunner
```

GitLab looks for a runner with the tag:

```text
localrunner
```

Your runner gets selected.

---

# 5. What happens in the `validate` stage?

This job:

```yaml
check_kubernetes:
  stage: validate

  tags:
    - localrunner

  script:
    - kubectl version --client
    - kubectl get nodes
    - kubectl get pods -A
```

executes:

```bash
kubectl version --client
```

Then:

```bash
kubectl get nodes
```

Then:

```bash
kubectl get pods -A
```

You should see your Kubernetes node.

For example:

```text
NAME      STATUS   ROLES           AGE
kmaster   Ready    control-plane   10d
```

---

# 6. Deploy stage

This creates a namespace:

```bash
kubectl create namespace gitlab-demo
```

Then creates an NGINX deployment:

```bash
kubectl create deployment nginx \
  --image=nginx:latest \
  -n gitlab-demo
```

The Kubernetes architecture becomes:

```text
gitlab-demo namespace
        |
        v
   nginx Deployment
        |
        v
    nginx Pod
        |
        v
 nginx:latest
```

Then we create a Service:

```bash
kubectl expose deployment nginx \
  --port=80 \
  --target-port=80 \
  --type=NodePort \
  -n gitlab-demo
```

So:

```text
NodePort Service
       |
       v
   nginx Service
       |
       v
 nginx Pod :80
```

---

# 7. Verify stage

The pipeline executes:

```bash
kubectl get deployment -n gitlab-demo
```

Then:

```bash
kubectl get pods -n gitlab-demo
```

Then:

```bash
kubectl get service -n gitlab-demo
```

And finally:

```bash
kubectl rollout status deployment/nginx \
  -n gitlab-demo \
  --timeout=120s
```

You should eventually see:

```text
deployment "nginx" successfully rolled out
```

---

# 8. Cleanup stage

The final stage removes everything:

```bash
kubectl delete namespace gitlab-demo
```

Because deleting the namespace also deletes resources inside it:

```text
gitlab-demo
   |
   +-- Deployment
   |
   +-- Pod
   |
   +-- Service
```

all get removed.

### If you want to keep the application

For your first test, I recommend keeping cleanup because it prevents your cluster from accumulating test resources.

Later, when you start building a real CI/CD deployment pipeline, remove the cleanup stage.

---

# 9. First test without GitLab

Before pushing the `.gitlab-ci.yml`, test exactly what GitLab Runner will execute.

Run:

```bash
sudo -u gitlab-runner kubectl get nodes
```

Then:

```bash
sudo -u gitlab-runner kubectl create namespace gitlab-demo
```

Then:

```bash
sudo -u gitlab-runner kubectl create deployment nginx \
  --image=nginx:latest \
  -n gitlab-demo
```

Check:

```bash
sudo -u gitlab-runner kubectl get pods -n gitlab-demo
```

If that works, your GitLab pipeline should also be able to access Kubernetes.

Clean it:

```bash
sudo -u gitlab-runner kubectl delete namespace gitlab-demo
```

---

# 10. One important consideration for your setup

You said:

> Kubernetes is installed on this system and this is my kmaster system

So you're currently doing:

```text
GitLab
   |
   | Internet
   v
kmaster
   |
   +-- GitLab Runner
   |
   +-- kubectl
   |
   +-- Kubernetes Control Plane
```

This is actually a **very good lab setup** for learning GitLab CI/CD + Kubernetes.

You don't need a separate Kubernetes deployment server initially.

The runner itself can execute:

```bash
kubectl apply
kubectl get
kubectl rollout
kubectl delete
```

against the local Kubernetes cluster.

Later, you can move toward the more production-like architecture:

```text
                    GitLab
                       |
                 CI/CD Pipeline
                       |
                       v
                 GitLab Runner
                       |
                       | kubectl / Helm
                       v
              Kubernetes Cluster
              +------------------+
              |                  |
         Control Plane        Workers
              |                  |
           kmaster             kworker
```

For your **next GitLab + Kubernetes lesson**, I would recommend changing this from `kubectl create` commands to **Kubernetes YAML manifests + `kubectl apply`**, so that your GitLab repository contains something like:

```text
project/
│
├── .gitlab-ci.yml
│
└── k8s/
    ├── namespace.yaml
    ├── deployment.yaml
    └── service.yaml
```

That will give you the proper **GitLab CI → Kubernetes deployment** pattern you'll use in real projects.
