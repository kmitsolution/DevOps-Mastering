# Lesson 2 — Install Argo CD on Kubernetes

In Lesson 1, we learned:

```text
GitLab
   ↓
Desired State
   ↓
Argo CD
   ↓
Kubernetes
```

Now we'll actually **install Argo CD into your Kubernetes cluster**.

---

## 1. What are we going to do?

In this lesson you'll:

1. Verify the Kubernetes cluster
2. Verify the `argocd` namespace
3. Install Argo CD
4. Understand the components Argo CD creates
5. Verify Argo CD pods
6. Access the Argo CD UI
7. Get the initial admin password
8. Log in to Argo CD

---

# Step 1 — Verify Your Kubernetes Cluster

Run:

```bash
kubectl get nodes
```

You should see:

```text
NAME       STATUS   ROLES           AGE
master     Ready    control-plane   ...
worker1    Ready    <none>          ...
worker2    Ready    <none>          ...
```

All nodes should preferably be:

```text
Ready
```

Now:

```bash
kubectl get pods -A
```

Make sure your cluster is generally healthy before installing Argo CD.

---

# Step 2 — Verify the Argo CD Namespace

We created this in Lesson 1.

Run:

```bash
kubectl get namespace argocd
```

Expected:

```text
NAME      STATUS   AGE
argocd    Active   ...
```

If you didn't create it, run:

```bash
kubectl create namespace argocd
```

---

# Step 3 — Install Argo CD

Now the important command.

Run:

```bash
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

You should see many resources being created:

```text
customresourcedefinition.apiextensions.k8s.io/... created
serviceaccount/... created
role.rbac.authorization.k8s.io/... created
deployment.apps/... created
service/... created
configmap/... created
secret/... created
```

Don't worry about all these resources yet.

We'll understand them.

---

# Step 4 — Check Argo CD Pods

Run:

```bash
kubectl get pods -n argocd
```

Initially, you may see:

```text
NAME                                      READY   STATUS
argocd-application-controller-xxxxx      0/1     ContainerCreating
argocd-repo-server-xxxxx                  0/1     ContainerCreating
argocd-server-xxxxx                      0/1     ContainerCreating
```

Wait a little and run again:

```bash
kubectl get pods -n argocd
```

Eventually you should see components in:

```text
Running
```

For example:

```text
NAME                                             READY   STATUS
argocd-application-controller-xxxx              1/1     Running
argocd-applicationset-controller-xxxx           1/1     Running
argocd-dex-server-xxxx                           1/1     Running
argocd-notifications-controller-xxxx            1/1     Running
argocd-redis-xxxx                                1/1     Running
argocd-repo-server-xxxx                          1/1     Running
argocd-server-xxxx                               1/1     Running
```

The exact names will contain random characters.

---

# Step 5 — Understand What Was Installed

This is important.

Don't think:

> "Argo CD is one pod."

It isn't.

Argo CD is a collection of Kubernetes components.

The major components are:

```text
                    Argo CD
                       |
        +--------------+--------------+
        |              |              |
        v              v              v
   API Server      Repo Server    Controller
        |              |              |
        |              |              |
       UI           GitLab         Kubernetes
```

Let's understand the important ones.

---

# Step 6 — Argo CD Server

Pod:

```text
argocd-server
```

This provides the main Argo CD API and UI.

You interact with Argo CD through:

```text
Browser
   |
   v
Argo CD Server
```

It also handles communication from:

* Argo CD CLI
* UI
* API clients

---

# Step 7 — Argo CD Repository Server

Pod:

```text
argocd-repo-server
```

This component communicates with Git repositories.

For our future setup:

```text
GitLab
   |
   v
Repo Server
```

It retrieves things such as:

```text
deployment.yaml
service.yaml
kustomization.yaml
Helm charts
```

and generates the Kubernetes manifests that Argo CD needs.

---

# Step 8 — Application Controller

Pod:

```text
argocd-application-controller
```

This is one of the **most important components**.

Remember our Lesson 1 concept:

```text
Desired State
      vs
Live State
```

The controller performs this comparison.

Conceptually:

```text
              Controller
                  |
       +----------+----------+
       |                     |
       v                     v
     Git                  Kubernetes
  Desired State            Live State
       |                     |
       +----------+----------+
                  |
                  v
             Difference?
```

If there's a difference:

```text
OUT OF SYNC
```

The controller can then reconcile the application depending on your sync configuration.

---

# Step 9 — ApplicationSet Controller

You'll encounter:

```text
argocd-applicationset-controller
```

We won't use it yet.

Later we'll use it to automatically generate multiple Argo CD Applications.

For example:

```text
ApplicationSet
     |
     +-- dev
     +-- test
     +-- qa
     +-- prod
```

We'll cover this much later.

---

# Step 10 — Redis

You'll see:

```text
argocd-redis
```

Argo CD uses Redis for internal caching.

You don't normally interact with Redis directly during normal Argo CD usage.

---

# Step 11 — Dex

You may see:

```text
argocd-dex-server
```

Dex is related to authentication/identity integration.

Later, when we cover enterprise Argo CD, we'll discuss integrations such as:

```text
LDAP
OIDC
SSO
```

Don't worry about Dex for now.

---

# Step 12 — Notifications Controller

You'll also see:

```text
argocd-notifications-controller
```

It can be used to send notifications about application events.

For example:

```text
Application Synced
Application Failed
Application Degraded
```

Later we can connect this to systems such as Slack or email.

---

# Step 13 — See All Argo CD Services

Run:

```bash
kubectl get svc -n argocd
```

You should see services similar to:

```text
NAME                          TYPE        CLUSTER-IP
argocd-applicationset-controller
argocd-dex-server
argocd-metrics
argocd-notifications-controller-metrics
argocd-redis
argocd-repo-server
argocd-server
argocd-server-metrics
```

Notice something important:

```text
argocd-server
```

is normally exposed as:

```text
ClusterIP
```

That means it isn't directly accessible from your Windows host.

For our lab, we'll use **port forwarding**.

---

# Step 14 — Access Argo CD UI

Run:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

You should see:

```text
Forwarding from 127.0.0.1:8080 -> 443
```

Keep this terminal running.

Now open your browser:

```text
https://localhost:8080
```

You may receive a browser warning about the certificate.

That's expected in this local lab.

Proceed to the Argo CD login page.

---

# Step 15 — Get the Initial Admin Password

Open another terminal.

Run:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
```

You'll get a password.

For example:

```text
xxxxxxxxxxxxxxxxxxxxxxxx
```

Your username is:

```text
admin
```

So:

```text
Username: admin
Password: <generated-password>
```

---

# Step 16 — Login to Argo CD

Go to:

```text
https://localhost:8080
```

Enter:

```text
Username:
admin
```

and the password you retrieved.

You should now see the Argo CD dashboard.

Initially, you probably won't have any applications.

That's perfectly normal.

You have installed the **Argo CD platform**, but we haven't created an Argo CD Application yet.

---

# Step 17 — Understand an Important Distinction

This is something beginners often misunderstand.

Installing Argo CD:

```text
Argo CD installed
```

does **NOT** mean:

```text
Your application deployed
```

We currently have:

```text
Kubernetes
   |
   +-- argocd namespace
          |
          +-- Argo CD components
```

But we don't yet have:

```text
Argo CD Application
```

Later we'll create:

```text
Argo CD Application
       |
       v
GitLab Repository
       |
       v
Kubernetes Resources
```

---

# Step 18 — Check Argo CD CRDs

Argo CD introduces Kubernetes Custom Resources.

Run:

```bash
kubectl get crd | grep argoproj
```

You should see resources such as:

```text
applications.argoproj.io
applicationsets.argoproj.io
appprojects.argoproj.io
```

These are extremely important.

Argo CD extends Kubernetes using **Custom Resource Definitions (CRDs)**.

For example:

```text
Application
ApplicationSet
AppProject
```

are Kubernetes resources understood by Argo CD.

---

# Step 19 — Check the Argo CD Namespace

Run:

```bash
kubectl get all -n argocd
```

This lets you see the resources Argo CD installed.

You can also run:

```bash
kubectl get deployments -n argocd
```

and:

```bash
kubectl get services -n argocd
```

---

# Step 20 — Lesson 2 Architecture

At this point your cluster should conceptually look like:

```text
                Kubernetes Cluster
                       |
                +------+------+
                |             |
                v             v
            argocd        Other namespaces
                |
      +---------+----------+
      |         |          |
      v         v          v
   Server    Repo Server Controller
      |         |          |
      |         |          |
      v         v          v
     UI       GitLab    Kubernetes
```

The most important components to remember are:

| Component                          | Purpose                            |
| ---------------------------------- | ---------------------------------- |
| `argocd-server`                    | UI/API/CLI                         |
| `argocd-repo-server`               | Communicates with Git repositories |
| `argocd-application-controller`    | Compares desired vs live state     |
| `argocd-applicationset-controller` | Generates Applications             |
| `argocd-redis`                     | Internal caching                   |
| `argocd-dex-server`                | Authentication/identity            |
| Notifications controller           | Application notifications          |

---

# Lesson 2 Exercise

Run these commands and verify the output:

```bash
kubectl get pods -n argocd
```

```bash
kubectl get svc -n argocd
```

```bash
kubectl get crd | grep argoproj
```

Then start the port-forward:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Open:

```text
https://localhost:8080
```

and log in with:

```text
admin
```

plus the password retrieved using:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
```

### Your goal for Lesson 2

You should finish with:

```text
✅ Argo CD installed
✅ All Argo CD pods Running
✅ Argo CD UI accessible
✅ Logged into Argo CD
✅ Understand Server
✅ Understand Repo Server
✅ Understand Application Controller
✅ Understand Argo CD CRDs
```

**Next: Lesson 3 — Create your first GitLab repository and deploy your first Kubernetes application through Argo CD.**
