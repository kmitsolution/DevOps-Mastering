# DAST tools

This is a classroom-ready hands-on lab you can demonstrate to students. We'll use OWASP ZAP, an open-source web application security testing tool, to scan a deliberately vulnerable application running in Docker. Then we'll look at how the same approach works with Kubernetes.

Students will learn how to:

- Run a web application in a container.
- Scan a running application using ZAP.
- Read and explain DAST findings.
- Fix a security header issue and rescan.
- Understand how DAST fits into a CI/CD pipeline.
- Adapt the same workflow to Kubernetes.

Safety: Use the deliberately vulnerable training application only in an isolated lab. Do not expose it publicly or scan systems without authorization.

## Lab architecture

Student browser

Access the demo application

Docker web application

OWASP Juice Shop — intentionally vulnerable

OWASP ZAP

Crawl, passively inspect, and report security findings

We'll start with a ZAP Baseline Scan, which crawls the target and performs passive analysis. It is a safer introduction than immediately running active attack tests.

## Practical 1 — Run OWASP Juice Shop in Docker

OWASP Juice Shop is an intentionally vulnerable web application designed for security training. It includes common web security flaws and is suitable for practicing DAST.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://devguide.owasp.org\&sz=32)

OWASP Developer Guide

+1



### Step 1: Start the application

On your Ubuntu machine, run:

```
docker pull bkimminich/juice-shop

docker run -d \
  --name juice-shop \
  -p 127.0.0.1:3000:3000 \
  bkimminich/juice-shop
```

The port is bound to `127.0.0.1`, so the application is accessible only from your own machine by default.

### Step 2: Verify the application

```
docker ps
curl -I http://localhost:3000
```

Open this URL in your browser:

`http://localhost:3000`

You should see the Juice Shop web application.

### Step 3: Run the ZAP Baseline Scan

Create a directory for reports:

```
mkdir -p "$HOME/dast-lab/reports"
```

Run the scanner in another container:

```
docker run --rm \
  -v "$HOME/dast-lab/reports:/zap/wrk/:rw" \
  ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py \
  -t http://host.docker.internal:3000 \
  -m 2 \
  -r juice-shop-report.html
```

On Linux, `host.docker.internal` may not resolve automatically. If the scan cannot connect, rerun it with the host gateway configured:

```
docker run --rm \
  --add-host=host.docker.internal:host-gateway \
  -v "$HOME/dast-lab/reports:/zap/wrk/:rw" \
  ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py \
  -t http://host.docker.internal:3000 \
  -m 2 \
  -r juice-shop-report.html
```

The `-m 2` option limits the spidering phase to approximately two minutes; the complete scan may take longer. ZAP's baseline scan performs passive analysis rather than the active attack scan.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.zaproxy.org\&sz=32)

ZAP Docker Documentation

+1



### Step 4: Review the report

Open the generated report:

```
xdg-open "$HOME/dast-lab/reports/juice-shop-report.html"
```

Discuss the following with students:

- What is the alert name?
- Which URL triggered it?
- What evidence did ZAP report?
- Is the finding a confirmed vulnerability or a potential issue?
- What fix would you recommend?
- How would you verify the fix with another scan?

Remember: a baseline scan may report warnings without failing the scan. Its exit code and the actual findings must be interpreted separately.

## Practical 2 — Understand a Burp Suite-style testing workflow

Burp Suite and ZAP share a similar core workflow: intercept or observe HTTP traffic, inspect requests and responses, identify potential vulnerabilities, and validate findings.

1\. Browse the application

Open Juice Shop and navigate through pages, search, and forms using test data.

2\. Inspect HTTP requests

Observe the URL, method, headers, cookies, query parameters, and response body through the tool's proxy or browser integration.

3\. Investigate a finding

Use the request and response evidence to understand why the scanner raised an alert. Reproduce only within the authorized lab.

4\. Document and retest

Record the affected endpoint, risk, remediation, and retest result.

Classroom exercise: Ask students to identify a missing security header in the ZAP report, check the actual HTTP response with `curl -I`, and explain how the web server configuration could be corrected. This teaches them to validate a finding instead of blindly accepting scanner output.

## Practical 3 — Run DAST against an application in Kubernetes

Now let's deploy the same training application to Kubernetes and scan it through a Kubernetes Service.

This exercise works on Minikube, kind, or another cluster where you have permission to create workloads.

### Step 1: Deploy Juice Shop

Create a file named `juice-shop.yaml`:

```

apiVersion: apps/v1
kind: Deployment
metadata:
  name: juice-shop
spec:
  replicas: 1
  selector:
    matchLabels:
      app: juice-shop
  template:
    metadata:
      labels:
        app: juice-shop
    spec:
      containers:
        - name: juice-shop
          image: bkimminich/juice-shop
          ports:
            - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: juice-shop
spec:
  type: ClusterIP
  selector:
    app: juice-shop
  ports:
    - port: 3000
      targetPort: 3000

```

Apply it:

```
kubectl apply -f juice-shop.yaml
kubectl rollout status deployment/juice-shop
kubectl get pods,svc
```

### Step 2: Forward the Service port

In terminal 1, run:

```
kubectl port-forward service/juice-shop 3000:3000
```

Keep that terminal open. The application should now be available at `http://localhost:3000` on your Kubernetes administration machine.

### Step 3: Scan the Kubernetes application

In terminal 2, run:

```
mkdir -p "$HOME/dast-lab/k8s-reports"

docker run --rm \
  --network host \
  -v "$HOME/dast-lab/k8s-reports:/zap/wrk/:rw" \
  ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py \
  -t http://127.0.0.1:3000 \
  -m 2 \
  -r kubernetes-dast-report.html
```

This example uses Linux host networking so the ZAP container can reach the port forwarded on the Ubuntu host. It is intended for a Linux machine. If you use Docker Desktop or a remote Kubernetes cluster, the network configuration will differ.

Review the report:

```
xdg-open "$HOME/dast-lab/k8s-reports/kubernetes-dast-report.html"
```

### Step 4: Clean up the lab

After the class, remove the training workload:

```
kubectl delete -f juice-shop.yaml
```

For the Docker-only exercise, stop and remove Juice Shop:

```
docker stop juice-shop
docker rm juice-shop
```

## 4. How to choose the right DAST tool

| Requirement                           | Suitable approach                                                      |
| ------------------------------------- | ---------------------------------------------------------------------- |
| Quick scan of a local Docker app      | ZAP Baseline Scan                                                      |
| Interactive request/response analysis | ZAP Desktop or Burp Suite                                              |
| API testing                           | ZAP API Scan using an OpenAPI definition                               |
| Kubernetes staging test               | Deploy the app, expose it only within the approved scope, then run ZAP |
| Automated CI/CD checks                | ZAP Docker scans or its Automation Framework                           |

ZAP also provides a dedicated API scan for APIs described using OpenAPI or GraphQL, which is useful for a .NET API lab.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://www.zaproxy.org\&sz=32)

ZAP Docker Documentation

+1



## Suggested classroom assignment

Ask each student to complete the following:

- Deploy Juice Shop in Docker.
- Run ZAP Baseline Scan.
- Save and explain at least three findings.
- Verify one finding using an HTTP request or response.
- Deploy Juice Shop to Kubernetes.
- Run the scan again and compare the reports.
- Clean up all test resources.

The main lesson for students is that DAST tests the running application's observable behavior. Docker and Kubernetes provide deployment environments; ZAP or Burp performs the security testing.
