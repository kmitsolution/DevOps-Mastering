
### 1. A DevOps engineer is designing a system to restrict EC2 access to specific roles only. What is the best enforcement method?

```
A. Use Security Groups to enforce access to required EC2 roles 
B. Use Config conformance pack to check for role-based access
C. Use IAM policies with specific role ARNs in resource blocks 
D. Use GuardDuty to detect access to EC2 from unknown roles
```

---

### 2. You want to enforce encryption on all S3 buckets while preventing public access. What is the most effective solution?

```
A. Use IAM deny policy for any S3 put-object operation
B. Use AWS Config rules to detect and block non-compliant buckets
C. Use CloudTrail logs to generate alerts for public access issues
D. Use KMS key policy to block public access to stored objects
```

---

### 3. An audit requires all EBS volumes to be encrypted at launch. What is the correct strategy to enforce this?

```
A. Enable GuardDuty to alert for unencrypted EBS volumes 
B. Use CloudTrail to track unencrypted volume creations
C. Use IAM deny policies to block unencrypted EBS launches
D. Use AWS Config to enforce EBS encryption via rules
```

---

### 4. A company wants to ensure credentials that haven't been used for 90 days are revoked. What tool helps with this?

```
A. Use IAM Credential Reports and Lambda to deactivate stale access keys
B. Use CloudTrail to restrict credentials older than 90 days 
C. Use Config to delete users with old keys
D. Use GuardDuty to identify users with expired credentials
```

---

### 5. To protect sensitive data, your organization requires encryption using a customer-supplied key. Which solution is suitable?

```
A. Use S3 default encryption with AWS-managed CMKS
B. Use SSE-C with S3 and provide encryption keys with each request
C. Use SSE-S3 with enforced KMS key policy
D. Use CloudTrail to log encryption operations across S3
```

---

### 6. Your team needs to ensure all RDS databases are encrypted. What is the best method for enforcing this?

```
A. Use IAM deny policies on unencrypted RDS resources 
B. Use GuardDuty to find unencrypted database instances
C. Use AWS Config rules to detect unencrypted RDS deployments
D. Use CloudTrail logs to find RDS snapshots without encryption
```

---

### 7. Which AWS service can be used to continuously detect unintended resource access configurations?

```
A. Use IAM Access Analyzer to detect external access to resources
B. Use CloudTrail to detect identity-based access issues 
C. Use GuardDuty to notify on overly broad IAM policies 
D. Use Inspector to audit security group exposure
```

---

### 8. What is the best way to prevent accidental deletion of KMS keys?

```
A. Use IAM deny statements for DeleteKey operation
B. Enable key deletion protection in AWS KMS key settings
C. Use CloudTrail to log deletion events and send alarms
D. Use GuardDuty to monitor key deletion attempts
```

---

### 9. Which AWS service evaluates compliance using prepackaged controls like CIS benchmarks?

```
A. AWS Audit Manager with built-in frameworks for control mapping
B. AWS Config with custom Lambda functions for benchmarking
C. Amazon Inspector with compliance scorecards 
D. CloudTrail with Athena queries for control analysis
```

---

### 10. Your company must prevent IAM users from changing their own permissions. What approach should you take?

```
A. Use Config to detect and delete custom policies
B. Use IAM Access Analyzer to detect self-modifying policies 
C. Use IAM permission boundaries to restrict user escalation 
D. Use CloudTrail logs to track policy change attempts
```

---

### 11. A security team wants to get notified immediately when root credentials are used. What is the best approach?

```
A. Use CloudWatch Events and Lambda for root login detection
B. Use IAM policy to deny root access to the Console 
C. Use AWS Inspector to flag root login behavior
D. Use Config conformance pack to evaluate root activity
```

---

### 12. You want to audit all instances of unencrypted data stored in S3 buckets. What service can help automate this?

```
A. AWS Shield for data encryption audits
B. AWS Inspector for object-level data scans
C. Amazon Macie to discover and alert on unencrypted sensitive data
D. IAM Policy Simulator to detect unsecured objects
```

---

### 13. A compliance officer wants to validate that security controls are consistently applied across regions. What should be used?

```
A. Use IAM roles replicated across all regions
B. Use AWS Config aggregator to consolidate findings across regions
C. Use GuardDuty to scan all regions for issues
D. Use KMS to restrict encryption in specific regions
```

---

### 14. You want to scan container images for vulnerabilities before deployment. Which service should you use?

```
A. Amazon Macie to review container code
B. Amazon Inspector to scan ECR images for security issues
C. AWS Config to enforce container health policies
D. CloudTrail to monitor container runtime errors
```

---

### 15. What AWS service allows automation of policy enforcement and remediation based on compliance drift?

```
A. AWS Config with custom remediation actions or SSM automation
B. Amazon Macie with default encryption rules
C. CloudTrail using Athena queries to filter policy violations
D. Inspector to remove non-compliant resources automatically
```

---

### 16. How can you prevent IAM users from disabling CloudTrail logging?

```
A. Use GuardDuty to block unauthorized activity 
B. Use Config to enable CloudTrail logs across regions 
C. Use SCPs to deny cloudtrail:StopLogging to all users 
D. Use Macie to block actions on CloudTrail logs
```

---

### 17. To ensure consistent encryption policies, what should you implement across all new RDS snapshots?

```
A. Use Config rules to require snapshot encryption at creation time
B. Use IAM deny statements on unencrypted snapshot sharing 
C. Use GuardDuty to detect shared snapshot misconfigurations 
D. Use KMS key aliases for all unencrypted snapshot policies
```

---

### 18. Your team wants to enforce encryption of all new Amazon SQS queues in all environments. What is the correct approach?

```
A. Use IAM deny policies for non-encrypted queue creation
B. Use AWS Shield to monitor unencrypted SQS traffic flows
C. Use GuardDuty to detect queues without key encryption 
D. Use AWS Config rules to require encryption on all queues
```

---

### 19. What method allows organizations to audit and analyze permissions granted via IAM policies?

```
A. Use Macie to analyze policy exposure and changes
B. Use CloudTrail to record all policy creation attempts
C. Use IAM Access Analyzer to review and simulate policies 
D. Use Inspector to detect overly permissive IAM attachments
```

---

### 20. An administrator needs to prevent IAM users from launching EC2 instances with public IPs. What's the best control?

```
A. Use GuardDuty to monitor EC2 IP traffic changes
B. Use AWS Config rules to deny instances with public IPs
C. Use KMS to encrypt IP address settings in configurations 
D. Use IAM conditions to block launch with public addresses
```

---

# Answers

```
1. C
2. B
3. D
4. A
5. B
6. C
7. A
8. B
9. A
10. C
11. A
12. C
13. B
14. B
15. A
16. C
17. A
18. D
19. C
20. B
```

