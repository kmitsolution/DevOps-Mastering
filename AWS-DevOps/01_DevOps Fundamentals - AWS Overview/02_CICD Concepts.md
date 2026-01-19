
**CI/CD (Continuous Integration / Continuous Delivery or Deployment)** is an automated pipeline that takes code from **commit → production** safely, repeatedly, and with minimal human intervention.

![ChatGPT Image Jan 19, 2026, 06_58_34 AM](https://github.com/user-attachments/assets/06737c71-6042-4327-a740-b63f0c268464)

### Why AWS Emphasizes CI/CD

AWS designs for:

* Frequent, small changes
* Automation over manual work
* Fast recovery from failure
* Consistent environments

CI/CD is the **core DevOps mechanism** that enforces these principles.

---

## Continuous Integration (CI)

**What it is:**
Developers frequently merge code into a shared repository where builds and tests run automatically.

**AWS Implementation**

* Source: CodeCommit / GitHub
* Build & test: CodeBuild
* Artifacts: S3 / CodeArtifact

**Key Outcomes**

* Early defect detection
* Consistent builds
* Reduced integration risk

---

## Continuous Delivery vs Continuous Deployment

| Type                  | Meaning                                  | AWS Context                     |
| --------------------- | ---------------------------------------- | ------------------------------- |
| Continuous Delivery   | Deployable at any time (manual approval) | Used for regulated environments |
| Continuous Deployment | Automatically deploy to prod             | Used for mature DevOps teams    |

---

# CI/CD Stages → AWS Well-Architected Pillars Mapping

---

## 1. Source Stage (Code Commit)

**What Happens**

* Code commit
* Pull request
* Version control

### Pillars Supported

* **Operational Excellence**
* **Security**

### Why

* Version control enables auditability
* PR reviews enforce standards
* Infrastructure and app changes are tracked

**Exam Insight**
✔ Version-controlled IaC
❌ Direct console changes

---

## 2. Build Stage (Compile & Package)

**What Happens**

* Code compilation
* Dependency resolution
* Artifact creation
* Unit tests

### Pillars Supported

* **Operational Excellence**
* **Performance Efficiency**
* **Cost Optimization**

### Why

* Automated builds reduce human error
* Standardized artifacts improve performance consistency
* Short-lived build environments reduce cost

**Exam Insight**
✔ Managed build services
❌ Long-running build servers

---

## 3. Test Stage (Automated Testing)

**What Happens**

* Unit tests
* Integration tests
* Security scans
* Performance tests

### Pillars Supported

* **Reliability**
* **Security**
* **Performance Efficiency**

### Why

* Detect failures early
* Reduce production incidents
* Validate performance before release

**Exam Insight**
✔ Automated tests in pipeline
❌ Manual testing gates only

---

## 4. Deploy Stage (Release to Environment)

**What Happens**

* Deploy to staging or production
* Traffic shifting
* Health checks
* Rollbacks

### Pillars Supported

* **Reliability**
* **Operational Excellence**
* **Security**

### Why

* Blue/Green and Canary reduce blast radius
* Automated rollback lowers MTTR
* Secure deployment roles protect production

**Exam Insight**
✔ Blue/Green deployments
❌ In-place manual updates

---

## 5. Approval Stage (Optional)

**What Happens**

* Manual approval before production

### Pillars Supported

* **Security**
* **Operational Excellence**

### Why

* Compliance requirements
* Change control policies

**Exam Insight**
✔ Approval only when required
❌ Approval everywhere by default

---

## 6. Monitoring & Feedback Stage

**What Happens**

* Metrics and logs collection
* Alarm-based rollback
* Incident detection

### Pillars Supported

* **Reliability**
* **Performance Efficiency**
* **Cost Optimization**
* **Operational Excellence**

### Why

* Enables fast recovery
* Provides feedback for improvements
* Prevents over-provisioning

**Exam Insight**
✔ Automated alarms and rollback
❌ Reactive manual monitoring

---

## 7. Optimization & Improvement Loop

**What Happens**

* Analyze metrics
* Improve pipeline
* Optimize cost and performance

### Pillars Supported

* **All Pillars (Especially Sustainability)**

### Why

* Continuous improvement is core AWS philosophy
* Efficient pipelines reduce waste and cost

---

# CI/CD → Well-Architected Summary Table

| CI/CD Stage  | Well-Architected Pillars                      |
| ------------ | --------------------------------------------- |
| Source       | Operational Excellence, Security              |
| Build        | Operational Excellence, Performance, Cost     |
| Test         | Reliability, Security, Performance            |
| Deploy       | Reliability, Operational Excellence, Security |
| Approval     | Security, Operational Excellence              |
| Monitoring   | Reliability, Performance, Cost                |
| Optimization | All (incl. Sustainability)                    |

---

# AWS Exam Decision Rule (Very Important)

When choosing a CI/CD solution:

1. Prefer **managed services**
2. Prefer **automation**
3. Minimize **manual steps**
4. Reduce **blast radius**
5. Enable **fast rollback**

---

