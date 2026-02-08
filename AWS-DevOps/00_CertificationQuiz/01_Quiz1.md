# 🧪 AWS DevOps Mock Exam – Practice Test

Instructions:

 Choose the best answer for each question.
 Each question has one correct answer.
 Do not refer to the answer key until you complete all questions.

---

### Question 1
```
You are designing a CI/CD pipeline using AWS CodePipeline. You want to ensure that your build process runs automated tests and generates artifacts for deployment. Which service should you integrate for this step?

A. Use AWS CodeDeploy to execute integration test stages in the pipeline
B. Use AWS CodeBuild to compile source code and run unit tests
C. Use Amazon EC2 to compile the application and upload the build
D. Use AWS Lambda to automate deployment to each staging
```
---

### Question 2

```
You are using AWS CloudFormation in a multi-account setup. What is the best approach to deploy templates securely and consistently across accounts?

A. Deploy templates manually using the AWS Console in each account
B. Use StackSets with service-managed permissions for deployment
C. Use AWS CDK to compile and deploy to each account using IAM users
D. Copy templates across accounts using S3 and deploy with CLI

```
---

### Question 3

```
You are implementing blue/green deployments in AWS CodeDeploy for an EC2 application. What benefit does this approach offer over in-place deployments?

A. Reduces time spent on infrastructure provisioning during deployments
B. Allows real-time patching of instances without downtime or rollback
C. Enables zero-downtime deployments with easy rollback to previous state
D. Automates repository creation and tagging in version control systems

```
---

### Question 4
```
A DevOps team needs to centrally manage and rotate secrets (like API keys) used in their pipelines. What AWS service is best suited for this use case?

A. AWS Systems Manager Inventory
B. Amazon S3 with server-side encryption
C. Amazon CloudWatch Logs
D. AWS Secrets Manager

```
---

### Question 5
```
What is a key advantage of using AWS CDK over traditional CloudFormation YAML/JSON templates?

A. CDK allows defining infrastructure using familiar programming languages
B. CDK removes the need to define resource dependencies
C. CDK automatically provisions infrastructure across any cloud provider
D. CDK offers interactive wizards for building serverless applications
```
---

### Question 6
```
Which deployment strategy helps reduce risk during releases by gradually shifting production traffic and validating application behavior?

A. Blue/green deployments serving 100% traffic immediately
B. Canary deployments with incremental traffic shifting
C. Rolling deployments replacing instances randomly
D. All-at-once deployments across regions
```
---

### Question 7
```
A team wants to automatically patch EC2 instances across multiple AWS accounts. Which Systems Manager feature helps automate this securely?

A. Run Command with cross-account IAM roles
B. OpsCenter
C. Patch Manager with Resource Groups and shared documents
D. Session Manager
```
---

### Question 8
```
Which AWS service can store build artifacts for reuse in multiple stages of a CI/CD pipeline?

A. Amazon DynamoDB
B. Amazon S3
C. AWS Config
D. AWS IAM
```
---

### Question 9
```
You need to manage infrastructure as code in a hybrid setup using both AWS and on-premises resources. What IaC tool is most flexible?

A. AWS CloudFormation
B. AWS CDK
C. Terraform
D. Amazon ECS
```
---

### Question 10
```
What CodePipeline feature ensures that only approved changes progress to production?

A. Manual approval actions
B. IAM policies
C. CloudWatch alarms
D. CodeBuild environment variables
```
---

### Question 11
```
How does rolling deployment in AWS CodeDeploy work for an Auto Scaling group?

A. Replaces the entire Auto Scaling group at once
B. Switches DNS to a green environment
C. Replaces instances in batches
D. Updates all instances simultaneously
```
---

### Question 12
```
A DevOps engineer needs to execute test cases after each commit using AWS services. Which service fits this automation requirement?

A. AWS CodeDeploy
B. AWS CodeBuild
C. Amazon Inspector
D. AWS CodePipeline
```
---

### Question 13
```
You want to securely inject secrets into an EC2 instance during deployment. Which combination ensures security and automation?

A. EC2 user data with KMS
B. Lambda environment variables
C. S3 with pre-signed URLs
D. Systems Manager Parameter Store (SecureString) with IAM roles
```
---

### Question 14
```
Which Terraform feature helps manage reusable and shareable infrastructure definitions?

A. Variable interpolation
B. Local-exec provisioners
C. Terraform modules
D. Hardcoded resource values
```
---

### Question 15
```
Which benefit does using AWS Systems Manager Automation provide over custom scripts on EC2?

A. Launching baked AMIs
B. No IAM permissions required
C. Reusable, version-controlled workflows across accounts
D. Low-level network configuration
```
---

### Question 16
```
You are using AWS CloudFormation to deploy infrastructure across multiple regions. What is the most efficient way to automate this?

A. CLI scripts per region
B. Custom bash scripts in CodePipeline
C. CloudFormation StackSets
D. CodeDeploy hooks
```
---

### Question 17
```
You need to version and store container build artifacts for reuse across pipelines. Which AWS service should you use?

A. Amazon S3
B. AWS CodeBuild
C. Amazon ECR
D. Systems Manager Parameter Store
```
---

### Question 18
```
A company wants to perform automated tests before deploying to production. How should this be implemented in CodePipeline?

A. Lambda-based testing
B. Manual approval stage
C. CodeBuild test stage before production
D. CloudTrail validation
```
---

### Question 19
```
You are using Terraform to manage AWS infrastructure across accounts. How can you securely assume roles for cross-account deployment?

A. Multiple AWS profiles
B. AssumeRole in provider configuration
C. Long-term IAM keys in variables
D. EC2 metadata service
```
---

### Question 20
```
Which strategy allows near-zero-downtime deployment and instant rollback for ECS-based microservices?

A. Rolling deployments
B. AWS Batch deployments
C. All-at-once deployments
D. Blue/green deployments with CodeDeploy
```
### Question 21

```
You need to inject environment-specific secrets into your Lambda function securely during deployment. What is the best approach?

A. Store secrets in S3 and fetch them inside the function handler code
B. Use CloudWatch Events to set secrets as environment variables
C. Pass secrets as plain-text Lambda environment variables at deploy
D. Use Parameter Store SecureString and reference them via IAM
```
---

### Question 22
```
What CodeBuild feature allows caching dependencies between builds to reduce runtime?

A. Enable EC2 compute optimization in the CodeBuild settings
B. Use Lambda to preinstall dependencies before CodeBuild runs
C. Use custom CodePipeline scripts to cache dependencies manually
D. Enable local caching of directories like `/root/.npm` in buildspec
```
---

### Question 23
```
How does using AWS CDK enhance maintainability of infrastructure-as-code compared to raw CloudFormation?

A. Reduces need for templates by using hardcoded AWS CLI commands
B. Allows creating resource definitions directly using HTML templates
C. Enables modular, testable, and version-controlled codebase
D. Removes all complexity by automatically applying best practices
```
---

### Question 24
```
What is a secure and scalable way to manage patching of EC2 instances across multiple AWS accounts?

A. Use IAM cross-account access and write custom EC2 bash scripts
B. Use VPC peering and SSH to run patch commands across accounts
C. Use Systems Manager Patch Manager with resource tagging
D. Use EC2 Auto Recovery with lifecycle hooks to apply updates
```
---

### Question 25
```
Your organization needs to enforce consistent security policies for IaC across AWS accounts. What approach should be used?

A. Use EC2 scripts to enforce templates with restricted user access
B. Use service control policies with IAM to enforce IaC restrictions
C. Use CloudTrail to trigger alerts when restricted resources are created
D. Use CloudWatch metrics to auto-remediate non-compliant changes
```
---

### Question 26

```
Which method allows CodePipeline to trigger only when changes occur in a specific Git branch?

A. Use Lambda filters to intercept non-target branch events
B. Configure source action to monitor the exact Git branch
C. Add CloudWatch alarms that track commit logs manually
D. Use IAM policies to ignore non-matching branch triggers
```
---

### Question 27
```
Your team needs to automate environment provisioning using Terraform in multiple AWS accounts. What setup is best?

A. Use CLI profiles to hardcode credentials for each Terraform run
B. Use shared SSH keys with S3 buckets to separate environments
C. Use Terraform workspaces and `assume_role` per account
D. Use nested modules with hardcoded credentials in provider
```
---

### Question 28
```
You want to audit deployments and investigate changes across your CI/CD pipelines. Which AWS service should you enable?

A. Enable Trusted Advisor for advanced CI/CD audit capabilities
B. Use AWS Config to trigger alarms when CodePipeline runs fail
C. Enable CloudTrail to log actions across CodePipeline stages
D. Use CloudWatch dashboards to visualize changes in pipelines
```
---

### Question 29
```
Which approach enables automatic deployment rollback when CodeDeploy detects a failure?

A. Use manual rollback triggers configured in CodeDeploy console
B. Configure automatic rollback settings in the deployment group
C. Use Lambda to monitor and undo failed deployments
D. Set up S3 lifecycle rules to restore previous versions
```
---

### Question 30
```
Which AWS service helps detect and remediate drift in CloudFormation-managed resources?

A. Use AWS Config rules to manage drift across regions
B. Use CloudFormation drift detection for resource integrity
C. Use Systems Manager to inspect template discrepancies
D. Use Athena queries to track resource state changes
```
---

### Question 31
```
What is a key advantage of using artifact repositories such as CodeArtifact in CI/CD workflows?

A. Ensures direct dependency injection into Lambda functions
B. Helps compress and encrypt files before S3 upload
C. Enables versioned and secure dependency management
D. Allows mirroring of EC2 user data for deployments
```
---

### Question 32
```
How does using Systems Manager State Manager benefit multi-account infrastructure compliance?

A. Lets each instance store its configuration separately
B. Automates compliance enforcement using SSM documents
C. Grants CloudTrail ability to remediate configuration drift
D. Allows tagging of compliant EC2 instances for alerts
```
---

### Question 33
```
Which component of CodeBuild helps define the sequence of build and test commands?

A. Use IAM permissions to dictate execution flow
B. Use CloudTrail logs to define command ordering logic
C. Use Lambda scripts to set prebuild variables
D. Use `buildspec.yml` to define build instructions
```
---

### Question 34
```
You want to safely rotate secrets stored in Secrets Manager. Which strategy is most appropriate?

A. Use CodeDeploy scripts to store new secrets each week
B. Manually update secrets in the console and notify users
C. Enable automatic rotation with Lambda integration
D. Use IAM role chaining to rotate secrets by tagging
```
---

### Question 35
```
What feature in AWS CDK supports creating infrastructure that adapts based on the environment?

A. Use context variables inside CloudFormation templates
B. Define YAML overrides to change settings per region
C. Use CDK context and conditional constructs for flexibility
D. Deploy separate templates with renamed stack names
```
---



## 🧾 Answer Key (Check After Completion)

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
21. D
22. D
23. C
24. C
25. B
26. B
27. C
28. C
29. B
30. B
31. C
32. B
33. D
34. C
35. C

---
