# 🧪 **AWS DevOps Mock Exam – Practice Test**

**Instructions:**

* Choose the **best answer** for each question.
* Each question has **one correct answer**.
* Do not refer to the answer key until you complete all questions.

---

### **Question 1**
```
You are designing a CI/CD pipeline using AWS CodePipeline. You want to ensure that your build process runs automated tests and generates artifacts for deployment. Which service should you integrate for this step?

A. Use AWS CodeDeploy to execute integration test stages in the pipeline
B. Use AWS CodeBuild to compile source code and run unit tests
C. Use Amazon EC2 to compile the application and upload the build
D. Use AWS Lambda to automate deployment to each staging
```
---

### **Question 2**

```
You are using AWS CloudFormation in a multi-account setup. What is the best approach to deploy templates securely and consistently across accounts?

A. Deploy templates manually using the AWS Console in each account
B. Use StackSets with service-managed permissions for deployment
C. Use AWS CDK to compile and deploy to each account using IAM users
D. Copy templates across accounts using S3 and deploy with CLI

```
---

### **Question 3**

```
You are implementing blue/green deployments in AWS CodeDeploy for an EC2 application. What benefit does this approach offer over in-place deployments?

A. Reduces time spent on infrastructure provisioning during deployments
B. Allows real-time patching of instances without downtime or rollback
C. Enables zero-downtime deployments with easy rollback to previous state
D. Automates repository creation and tagging in version control systems

```
---

### **Question 4**
```
A DevOps team needs to centrally manage and rotate secrets (like API keys) used in their pipelines. What AWS service is best suited for this use case?

A. AWS Systems Manager Inventory
B. Amazon S3 with server-side encryption
C. Amazon CloudWatch Logs
D. AWS Secrets Manager

```
---

### **Question 5**
```
What is a key advantage of using AWS CDK over traditional CloudFormation YAML/JSON templates?

A. CDK allows defining infrastructure using familiar programming languages
B. CDK removes the need to define resource dependencies
C. CDK automatically provisions infrastructure across any cloud provider
D. CDK offers interactive wizards for building serverless applications
```
---

### **Question 6**
```
Which deployment strategy helps reduce risk during releases by gradually shifting production traffic and validating application behavior?

A. Blue/green deployments serving 100% traffic immediately
B. Canary deployments with incremental traffic shifting
C. Rolling deployments replacing instances randomly
D. All-at-once deployments across regions
```
---

### **Question 7**
```
A team wants to automatically patch EC2 instances across multiple AWS accounts. Which Systems Manager feature helps automate this securely?

A. Run Command with cross-account IAM roles
B. OpsCenter
C. Patch Manager with Resource Groups and shared documents
D. Session Manager
```
---

### **Question 8**
```
Which AWS service can store build artifacts for reuse in multiple stages of a CI/CD pipeline?

A. Amazon DynamoDB
B. Amazon S3
C. AWS Config
D. AWS IAM
```
---

### **Question 9**
```
You need to manage infrastructure as code in a hybrid setup using both AWS and on-premises resources. What IaC tool is most flexible?

A. AWS CloudFormation
B. AWS CDK
C. Terraform
D. Amazon ECS
```
---

### **Question 10**
```
What CodePipeline feature ensures that only approved changes progress to production?

A. Manual approval actions
B. IAM policies
C. CloudWatch alarms
D. CodeBuild environment variables
```
---

### **Question 11**
```
How does rolling deployment in AWS CodeDeploy work for an Auto Scaling group?

A. Replaces the entire Auto Scaling group at once
B. Switches DNS to a green environment
C. Replaces instances in batches
D. Updates all instances simultaneously
```
---

### **Question 12**
```
A DevOps engineer needs to execute test cases after each commit using AWS services. Which service fits this automation requirement?

A. AWS CodeDeploy
B. AWS CodeBuild
C. Amazon Inspector
D. AWS CodePipeline
```
---

### **Question 13**
```
You want to securely inject secrets into an EC2 instance during deployment. Which combination ensures security and automation?

A. EC2 user data with KMS
B. Lambda environment variables
C. S3 with pre-signed URLs
D. Systems Manager Parameter Store (SecureString) with IAM roles
```
---

### **Question 14**
```
Which Terraform feature helps manage reusable and shareable infrastructure definitions?

A. Variable interpolation
B. Local-exec provisioners
C. Terraform modules
D. Hardcoded resource values
```
---

### **Question 15**
```
Which benefit does using AWS Systems Manager Automation provide over custom scripts on EC2?

A. Launching baked AMIs
B. No IAM permissions required
C. Reusable, version-controlled workflows across accounts
D. Low-level network configuration
```
---

### **Question 16**
```
You are using AWS CloudFormation to deploy infrastructure across multiple regions. What is the most efficient way to automate this?

A. CLI scripts per region
B. Custom bash scripts in CodePipeline
C. CloudFormation StackSets
D. CodeDeploy hooks
```
---

### **Question 17**
```
You need to version and store container build artifacts for reuse across pipelines. Which AWS service should you use?

A. Amazon S3
B. AWS CodeBuild
C. Amazon ECR
D. Systems Manager Parameter Store
```
---

### **Question 18**
```
A company wants to perform automated tests before deploying to production. How should this be implemented in CodePipeline?

A. Lambda-based testing
B. Manual approval stage
C. CodeBuild test stage before production
D. CloudTrail validation
```
---

### **Question 19**
```
You are using Terraform to manage AWS infrastructure across accounts. How can you securely assume roles for cross-account deployment?

A. Multiple AWS profiles
B. AssumeRole in provider configuration
C. Long-term IAM keys in variables
D. EC2 metadata service
```
---

### **Question 20**
```
Which strategy allows near-zero-downtime deployment and instant rollback for ECS-based microservices?

A. Rolling deployments
B. AWS Batch deployments
C. All-at-once deployments
D. Blue/green deployments with CodeDeploy
```
---

## 🧾 **Answer Key (Check After Completion)**

1. B
2. B
3. C
4. D
5. A
6. B
7. C
8. B
9. C
10. A
11. C
12. B
13. D
14. C
15. C
16. C
17. C
18. C
19. B
20. D

---
