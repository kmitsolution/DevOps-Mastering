## 1. How can you simplify and version changes to complex CloudFormation stacks?

```
A. Copy and paste each change manually into console templates
B. Use nested stacks and modular YAML structure
C. Use CloudTrail logs to track manual edits to templates
D. Upload templates as plaintext strings in S3
```

---

## 2. What is the purpose of a pre-traffic hook in a blue/green CodeDeploy deployment?

```
A. It verifies the user is logged in before launching new tasks
B. It adds alarms to test application metrics after deployment
C. It switches DNS before starting health checks on the new version
D. It runs validation tests before routing production traffic
```

---

## 3. What is a best practice for secrets handling in IaC workflows using Terraform?

```
A. Hardcode secrets in Terraform plan files for simplicity
B. Store secrets in EC2 user data for every deployment
C. Reference secrets from Parameter Store using data blocks
D. Write secrets directly into JSON-based Terraform templates
```

---

## 4. A team wants to implement continuous testing in CI/CD. What is the best place to run tests?

```
A. Add test logic to Lambda invocations after deploy step
B. Add CodeBuild actions after source and before deploy
C. Add CloudFormation stack creation as a test step
D. Use CloudTrail to monitor API calls for test success
```

---

## 5. How does AWS Systems Manager Session Manager enhance security over SSH access?

```
A. Encrypts logs stored in S3 during command sessions
B. Enables secure access without opening SSH ports
C. Allows direct S3 object access from EC2 via tunneling
D. Gives IAM policies permission to manage Linux firewalls
```

---

## 6. You want to implement change approval before applying a Terraform plan. What is the best DevOps solution?

```
A. Use Lambda to notify users and auto-approve changes
B. Use CloudTrail to enforce pre-change logging rules
C. Use CodePipeline manual approval between Terraform stages
D. Use EventBridge to delay changes until policy check completes
```

---

## 7. What CodeDeploy strategy is most efficient for updating thousands of EC2 instances with minimal service disruption?

```
A. Use blue/green deployment to update all instances instantly
B. Use all-at-once strategy to replace current infrastructure
C. Use rolling update strategy to batch EC2 instance upgrades
D. Use Lambda to rotate EC2 user data per deployment cycle
```

---

## 8. Which solution ensures consistent runtime environments across all CodeBuild projects?

```
A. Use custom Docker images stored in Amazon ECR repositories
B. Use Lambda functions to generate runtime scripts dynamically
C. Use EC2 instance metadata service to define runtime variables
D. Use S3 buckets to store runtime binaries for installation later
```

---

## 9. What is the best way to reduce risk in production deployments for a critical API service?

```
A. Use manual deployments during off-peak business hours
B. Use canary deployments to release gradually with monitoring
C. Use all-at-once deployments to reduce deployment cycle time
D. Use Lambda to roll back failed updates after full release
```

---

## 10. A DevOps engineer needs to automate infrastructure provisioning for new AWS accounts. What is the best solution?

```
A. Use CloudWatch to detect account creation and trigger setup
B. Use Step Functions to call CLI commands for resource creation
C. Use AWS Organizations with CloudFormation StackSets
D. Use EC2 instance bootstrap scripts to configure accounts
```

---

## 11. Which service is best for managing secure access credentials in a CI/CD pipeline?

```
A. Use IAM static keys embedded in buildspec YAML files
B. Use Lambda environment variables to store credentials
C. Use Secrets Manager with fine-grained IAM policies
D. Use Amazon S3 signed URLs to store secret credentials
```

---

## 12. What feature in CodePipeline allows re-running failed stages without restarting the entire pipeline?

```
A. Use CloudWatch Events to restart pipeline automatically
B. Use retry stage functionality in CodePipeline UI
C. Use CodeBuild hooks to skip successful stage outputs
D. Use Lambda polling to trigger partial pipeline execution
```

---

## 13. What should be used to centrally manage parameters like AMI IDs across multiple stacks?

```
A. Use nested stacks to copy values between CloudFormation stacks
B. Use EC2 user data scripts to propagate global parameters
C. Use Systems Manager Parameter Store with global references
D. Use Amazon Athena to query shared values between templates
```

---

## 14. How can developers prevent sensitive values from appearing in CodeBuild logs?

```
A. Use environment variables with plaintext echo commands
B. Use S3 encrypted folders to write log values during builds
C. Use EC2 SSH access to filter logs after builds complete
D. Use parameter masking in CodeBuild for sensitive variables
```

---

## 15. What is a benefit of using Terraform remote state with state locking in production?

```
A. Prevents sensitive outputs from being encrypted in the logs
B. Ensures CLI access is distributed between developer accounts
C. Prevents concurrent executions from corrupting state data
D. Allows multiple regions to use one resource configuration
```

---

## 16. Which solution enables securely rotating database credentials used in CodeBuild?

```
A. Save credentials as environment variables in buildspec file
B. Store credentials in Secrets Manager with rotation enabled
C. Store credentials in S3 and rotate manually using CLI scripts
D. Use EC2 metadata to inject passwords into the runtime image
```

---

## 17. You want to ensure only approved changes are applied by CloudFormation templates. What should you do?

```
A. Apply StackSets with scheduled tasks to enforce deployments
B. Use EC2 instance tagging to confirm all changes before apply
C. Use CloudFormation ChangeSets for review before execution
D. Use CodePipeline stages to simulate stack updates silently
```

---

## 18. How can you define reusable and tested infrastructure logic in AWS CDK?

```
A. Use AWS SDK to manually deploy stack components from code
B. Create CDK constructs that encapsulate repeatable logic
C. Write long-form templates and include JSON string fragments
D. Copy-paste generated CDK output between all stack folders
```

---

## 19. What method ensures a Terraform deployment applies changes only after approval?

```
A. Use auto-apply enabled in Terraform to confirm changes
B. Use Lambda function to check for plan drift in regions
C. Use scheduled EC2 tasks to run Terraform nightly builds
D. Use CI/CD with manual approval before apply stage
```

---

## 20. A company requires a centralized approach to run operational tasks across all AWS accounts. What service should they use?

```
A. Use VPC Peering to connect all instances and run shell scripts
B. Use CodePipeline to invoke Lambda in every target account
C. Use Systems Manager Automation across accounts and regions
D. Use S3 replication to copy logs and initiate remote commands
```

---

# ✅ Answer Keys

1. B
2. D
3. C
4. B
5. B
6. C
7. C
8. A
9. B
10. C
11. C
12. B
13. C
14. D
15. C
16. B
17. C
18. B
19. D
20. C

---

