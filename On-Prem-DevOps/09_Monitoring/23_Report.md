Yes. This is a good hands-on example for the **Failed Deployment → Automatic Rollback** runbook.

One important point first:

> `kubectl create -f deploy.yaml` only tells GitLab whether Kubernetes accepted the manifest. It does **not** tell GitLab whether the Deployment actually became healthy.

So we'll change your pipeline to use:

```bash
kubectl apply -f deploy.yaml
kubectl rollout status deployment/<deployment-name> --timeout=60s
```

If rollout fails, the job will:

1. Detect failure
2. Roll back
3. Verify rollback
4. Write the result to `/tmp/status.txt`
5. Include the GitLab build/pipeline number

---

# GitLab CI/CD Pipeline

Assuming your Deployment in `deploy.yaml` is called:

```yaml
metadata:
  name: demo-app
```

use:

```yaml
stages:
  - test
  - deploy

test_runner:
  stage: test

  tags:
    - localrunner

  script:
    - echo "GitLab Runner is working!"
    - hostname
    - whoami
    - echo "Hello world1"

deploy_app:
  stage: deploy

  tags:
    - localrunner

  script:
    - echo "Starting deployment..."
    - echo "Build number: $CI_PIPELINE_ID"

    # Deploy application
    - kubectl apply -f deploy.yaml

    # Wait for Kubernetes rollout
    - |
      if kubectl rollout status deployment/demo-app --timeout=60s; then
        echo "Deployment successful"

        echo "Build Number: $CI_PIPELINE_ID" > /tmp/status.txt
        echo "Deployment Status: SUCCESS" >> /tmp/status.txt
        echo "Rollback: NOT REQUIRED" >> /tmp/status.txt

      else
        echo "Deployment FAILED"
        echo "Starting rollback..."

        # Rollback
        kubectl rollout undo deployment/demo-app

        # Wait for rollback to complete
        if kubectl rollout status deployment/demo-app --timeout=60s; then
          echo "Rollback completed successfully"

          echo "Build Number: $CI_PIPELINE_ID" > /tmp/status.txt
          echo "Deployment Status: FAILED" >> /tmp/status.txt
          echo "Rollback: SUCCESSFUL" >> /tmp/status.txt
        else
          echo "Rollback FAILED"

          echo "Build Number: $CI_PIPELINE_ID" > /tmp/status.txt
          echo "Deployment Status: FAILED" >> /tmp/status.txt
          echo "Rollback: FAILED" >> /tmp/status.txt

          exit 1
        fi
      fi

    # Display status
    - echo "===== Deployment Status ====="
    - cat /tmp/status.txt
    - echo "============================="
```

---

# What happens now?

## Successful deployment

Suppose:

```text
deploy.yaml
     ↓
nginx:1.27
     ↓
Kubernetes
     ↓
Pods Running
     ↓
rollout status = SUCCESS
```

`/tmp/status.txt` becomes:

```text
Build Number: 125
Deployment Status: SUCCESS
Rollback: NOT REQUIRED
```

---

# Failed deployment

Now deliberately change your `deploy.yaml`:

```yaml
image: nginx:does-not-exist
```

Pipeline:

```text
GitLab
   ↓
kubectl apply
   ↓
Deployment accepted
   ↓
Pod
   ↓
ImagePullBackOff
   ↓
rollout status
   ↓
❌ TIMEOUT
   ↓
Rollback
```

The pipeline then executes:

```bash
kubectl rollout undo deployment/demo-app
```

and waits:

```bash
kubectl rollout status deployment/demo-app --timeout=60s
```

If successful:

```text
Build Number: 126
Deployment Status: FAILED
Rollback: SUCCESSFUL
```

---

# Very Important: Don't use `$CI_BUILD_NUMBER`

GitLab provides several predefined CI variables.

For your requirement, I recommend:

```bash
$CI_PIPELINE_ID
```

Example:

```text
Build Number: 126
```

If you specifically want the **pipeline sequence number**, you can also use:

```bash
$CI_PIPELINE_IID
```

For example:

```text
Build Number: 42
```

### Difference

```text
CI_PIPELINE_ID
```

= globally unique pipeline ID.

```text
CI_PIPELINE_IID
```

= project-level pipeline number.

For a training/demo runbook, I'd use:

```bash
$CI_PIPELINE_IID
```

because it looks like the familiar GitLab **#42** pipeline number.

So I'd actually change:

```yaml
echo "Build Number: $CI_PIPELINE_ID"
```

to:

```yaml
echo "Build Number: $CI_PIPELINE_IID"
```

---

# One more improvement — preserve the status file

There is an operational issue with `/tmp/status.txt`.

`/tmp` belongs to the **GitLab Runner machine**.

So the file:

```text
/tmp/status.txt
```

may disappear or be overwritten by another job.

If you want the status to be available after the pipeline finishes, use a GitLab artifact as well.

Add:

```yaml
  artifacts:
    when: always
    paths:
      - /tmp/status.txt
```

However, GitLab artifacts generally expect files within the project workspace, so the cleaner approach is:

```bash
cp /tmp/status.txt status.txt
```

and:

```yaml
  artifacts:
    when: always
    paths:
      - status.txt
```

Then your job can have:

```text
/tmp/status.txt
       ↓
status.txt
       ↓
GitLab Artifact
```

---

# Recommended production-style version

I would therefore use:

```yaml
stages:
  - test
  - deploy

test_runner:
  stage: test

  tags:
    - localrunner

  script:
    - echo "GitLab Runner is working!"
    - hostname
    - whoami
    - echo "Hello world1"


deploy_app:
  stage: deploy

  tags:
    - localrunner

  script:
    - echo "Starting deployment..."
    - echo "Pipeline Number: $CI_PIPELINE_IID"

    - kubectl apply -f deploy.yaml

    - |
      if kubectl rollout status deployment/demo-app --timeout=60s; then

        echo "Deployment successful"

        echo "Build Number: $CI_PIPELINE_IID" > /tmp/status.txt
        echo "Deployment Status: SUCCESS" >> /tmp/status.txt
        echo "Rollback: NOT REQUIRED" >> /tmp/status.txt

      else

        echo "================================"
        echo "DEPLOYMENT FAILED"
        echo "================================"

        echo "Starting automatic rollback..."

        kubectl rollout undo deployment/demo-app

        if kubectl rollout status deployment/demo-app --timeout=60s; then

          echo "Rollback completed successfully"

          echo "Build Number: $CI_PIPELINE_IID" > /tmp/status.txt
          echo "Deployment Status: FAILED" >> /tmp/status.txt
          echo "Rollback: SUCCESSFUL" >> /tmp/status.txt

        else

          echo "Rollback failed"

          echo "Build Number: $CI_PIPELINE_IID" > /tmp/status.txt
          echo "Deployment Status: FAILED" >> /tmp/status.txt
          echo "Rollback: FAILED" >> /tmp/status.txt

          exit 1

        fi

      fi

    - cp /tmp/status.txt status.txt
    - cat /tmp/status.txt

  artifacts:
    when: always
    paths:
      - status.txt
```

## Final flow

```text
             GitLab Pipeline #42
                    │
                    ▼
              kubectl apply
                    │
                    ▼
             Rollout Status
                    │
             ┌──────┴──────┐
             │             │
          SUCCESS        FAILED
             │             │
             ▼             ▼
          Continue      Rollback
                           │
                           ▼
                    Rollout Status
                           │
                    ┌──────┴──────┐
                    ▼             ▼
                 SUCCESS        FAILED
                    │             │
                    ▼             ▼
                 Record         Record
                 status         failure
                    │
                    ▼
             status.txt
                    │
                    ▼
             GitLab Artifact
```

This gives you a very realistic **CI/CD failed-deployment runbook implementation**: GitLab doesn't merely deploy; it verifies the rollout, automatically recovers to the previous revision when appropriate, and leaves an auditable status artifact.
