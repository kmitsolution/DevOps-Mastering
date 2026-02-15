
## 1. What is the best way to audit usage of customer master keys in KMS?

```
A. Use AWS Inspector to evaluate encryption key integrity
B. Use AWS Config to gather rotation status of each key
C. Use GuardDuty to report on KMS access and usage
D. Use CloudTrail to monitor all KMS API interactions
```

---

## 2. You are asked to prevent all users in dev accounts from disabling GuardDuty. What should you implement?

```
A. Use SCPs to deny the StopMonitoring API across accounts
B. Use IAM role conditions to prevent disabling threat detection
C. Use Config to re-enable GuardDuty if it is disabled manually
D. Use CloudTrail logs and alarms for unauthorized changes
```

---

## 3. Which service helps discover and protect sensitive data stored in S3?

```
A. AWS Shield using DDoS detection and S3 tagging
B. Amazon Macie using ML-based data classification tools
C. AWS Inspector with deep content inspection engines
D. AWS Config with preconfigured rule packs for S3
```

---

## 4. Your team wants to enforce encryption on all new DynamoDB tables. What strategy should you use?

```
A. Use AWS Config with rules for encryption at rest checks
B. Use IAM policies to block table creation without encryption
C. Use CloudTrail events and alarms to track table settings
D. Use Amazon Macie to audit DynamoDB storage encryption
```

---

## 5. To ensure real-time response to security findings, what should a DevOps team implement?

```
A. Use GuardDuty to scan logs and review daily reports
B. Use Config to compile a weekly compliance score report
C. Use Inspector to review IAM configurations at runtime
D. Use EventBridge to trigger actions from GuardDuty events
```

---

## 6. Which of the following enables you to track whether resource configurations match approved baselines?

```
A. IAM credential reports from the AWS Organizations console
B. AWS Config tracking drift from defined compliance rules
C. CloudTrail storing logs from all configuration activities
D. AWS KMS logs showing access to encrypted resources
```

---

## 7. What should you implement to verify if RDS snapshots are being shared outside the organization?

```
A. Enable GuardDuty to block all unauthorized snapshots
B. Use CloudTrail to monitor snapshot modifications only
C. Use IAM policies to enforce snapshot sharing restrictions
D. Use IAM Access Analyzer to detect external snapshot access
```

---

## 8. A customer must prevent accidental deletion of CloudTrail trails. What should be enforced?

```
A. Use IAM policies to restrict deletion to admin users
B. Use SCPs to deny cloudtrail:DeleteTrail for all accounts
C. Use GuardDuty alerts when trail changes are made
D. Use Config to enable auto-recreate for deleted trails
```

---

## 9. What solution allows audit teams to map AWS actions to compliance requirements?

```
A. AWS Audit Manager using control mapping frameworks
B. AWS Config to list all changes in resource policies
C. CloudTrail to generate logs of all policy violations
D. GuardDuty to detect compliance events from IAM users
```

---

## 10. You are tasked with enforcing the use of a specific KMS key for all S3 buckets. What strategy ensures compliance?

```
A. Use IAM roles to enforce usage of custom encryption keys
B. Use AWS Config to check for specific key ARN in encryption
C. Use CloudTrail to detect buckets not using that key
D. Use GuardDuty to prevent use of incorrect KMS keys
```

---

## 11. How do you prevent deletion of logs during active incident analysis?

```
A. Use S3 Object Lock feature to retain log files
B. Use CloudWatch export to immutable third-party store
C. Use S3 with compliance-mode Object Lock
D. Use Athena query protection to lock dataset changes
```

---

## 12. Which option ensures that snapshots are created after a critical change?

```
A. Use Config rule to trigger a manual snapshot process
B. Use CloudWatch alarm and Step Functions to initiate
C. Use EventBridge with Lambda to trigger snapshots
D. Use CloudTrail event with EventBridge and Lambda
```

---

## 13. How do you confirm who approved a sensitive change during an incident?

```
A. Use CloudTrail to review IAM group policies changed
B. Use Systems Manager approval workflow logs
C. Use Config to show resource tag updates for approver
D. Use EventBridge archive and query previous event tags
```

---

## 14. What enables timely notification of service outages impacting your account?

```
A. Use CloudWatch alarm on AWS error rates per region
B. Use CloudTrail Insight events with alarm threshold
C. Use EventBridge with AWS Health Dashboard events
D. Use AWS Config to track changes in service statuses
```

---

## 15. You want to block an IAM user who shows malicious behavior. What is the fastest approach?

```
A. Use GuardDuty alert and IAM deny policy assignment
B. Use Config with remediation Lambda to remove user
C. Use CloudTrail Insight to notify on unauthorized use
D. Use EventBridge with Lambda to revoke permissions
```

---

## 16. What AWS feature ensures cross-region visibility of critical audit logs?

```
A. Use CloudWatch cross-region dashboard and widgets
B. Use Athena federated queries to join regional logs
C. Use CloudTrail multi-region configuration setting
D. Use Config aggregator across regions to track changes
```

---

## 17. How do you stop repeated failed login attempts from the same IP address?

```
A. Use CloudTrail to block access via CLI only
B. Use GuardDuty with EventBridge to update WAF
C. Use Config to detect and reset access controls
D. Use X-Ray to verify client request path and timing
```

---

## 18. What helps detect unintentional deletion of a backup resource?

```
A. Use S3 versioning and manual review of file changes
B. Use EventBridge with CloudTrail and file digest match
C. Use CloudTrail to detect deletion API and alert
D. Use GuardDuty to track failed recovery operations
```

---

## 19. Which tool allows responders to perform real-time forensics securely?

```
A. Use Athena with federated security datasets enabled
B. Use Lambda with S3 snapshots streamed to forensic tool
C. Use Systems Manager Session Manager to inspect hosts
D. Use GuardDuty suppression rules to limit findings view
```

---

## 20. How do you ensure system restores automatically from a known state?

```
A. Use EC2 Image Builder to trigger instance restore
B. Use SSM Automation runbook triggered by event
C. Use Lambda to deploy resource state from template
D. Use EventBridge with snapshot rollback integration
```

---

# ✅ Answer Keys

1. D
2. A
3. B
4. A
5. D
6. B
7. D
8. B
9. A
10. B
11. C
12. D
13. B
14. C
15. D
16. C
17. B
18. C
19. C
20. B

---


