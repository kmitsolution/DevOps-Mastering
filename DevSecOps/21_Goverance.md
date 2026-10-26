# Governance Considerations and Limitations of AI Assistance in DevSecOps

When using AI features in GitLab Duo or other AI coding assistants, organizations need governance rules to ensure that AI improves developer productivity without weakening security, privacy, or compliance.

The key principle is: AI can assist with decisions, but accountable humans and automated controls must validate the results.

## 1. Governance considerations

1\. Data privacy and confidentiality

- Do not paste passwords, API keys, access tokens, customer records, or production secrets into AI prompts.
- Follow organizational rules for source-code sharing and AI data processing.
- Review the provider's data-retention, privacy, and training policies before enabling AI features.

2\. Human oversight and accountability

- Developers must review AI-generated code, tests, and remediation advice.
- Code owners and reviewers remain responsible for approving merge requests.
- Never merge a security-sensitive change solely because AI recommends it.

3\. Compliance and licensing

- Follow applicable laws, contractual obligations, and internal security policies.
- Check licensing and provenance where AI-generated code resembles third-party code.
- Keep evidence of required reviews, approvals, and security checks.

4\. Auditability and access control

- Restrict AI features according to approved roles and permissions.
- Record required approvals and security findings through the normal development workflow.
- Verify which AI features are enabled and what audit information your GitLab edition provides.

## 2. Limitations of AI assistance

| Limitation                  | Example in DevSecOps                                                                   | Recommended control                                                         |
| --------------------------- | -------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| Incorrect recommendations   | AI suggests an authentication fix that still allows unauthorized access.               | Review code and test access-control scenarios.                              |
| False positives             | AI flags a safe operation as vulnerable.                                               | Validate the finding before changing code.                                  |
| False negatives             | AI misses a SQL injection vulnerability.                                               | Use SAST, SCA, DAST, and manual review.                                     |
| Incomplete test coverage    | Generated tests cover valid inputs but miss malicious inputs.                          | Add negative, boundary, authorization, and abuse-case tests.                |
| Outdated or missing context | AI suggests a package version or API that does not fit your project.                   | Verify against official documentation and the actual project configuration. |
| Over-reliance on AI         | A developer accepts generated code without understanding it.                           | Require human review and technical justification.                           |
| Prompt injection            | Malicious instructions in code comments or repository files influence an AI assistant. | Treat repository content as untrusted and review proposed actions.          |
| Inconsistent results        | The same prompt produces different recommendations.                                    | Use repeatable automated checks and documented acceptance criteria.         |

## 3. Practical governance policy for your GitLab project

For your .NET 10 API and Angular application, you can adopt the following policy.

## AI usage checklist

Use this as a checklist when defining your project policy.

Never submit credentials, production secrets, or sensitive customer data to unapproved AI tools.

Review and understand all AI-generated code before committing it.

Verify AI-generated tests and run them in the CI/CD pipeline.

Require merge request approval according to the project's code-owner and security rules.

Run required SAST, dependency, secret, and container-image scans independently of AI.

Validate security fixes and document accepted or rejected high-risk recommendations.

Use approved AI tools and follow organizational privacy, licensing, and retention requirements.

0 of 7 policy items reviewed

## 4. How governance works in a DevSecOps pipeline

```
Developer uses AI assistance
          |
          v
Human reviews code and tests
          |
          v
Merge request approval
          |
          v
Automated CI/CD security checks
  - Unit and integration tests
  - SAST
  - Dependency scanning
  - Secret detection
  - Trivy image scanning
          |
          v
Policy checks pass?
     /          \
   No             Yes
   |               |
Fix and retest   Continue to deployment
```

AI assistance and automated security controls serve different purposes. AI helps developers create and review code; scanners and tests provide repeatable checks, while governance defines who may approve changes and what evidence is required.

## 5. What organizations should remember

- AI is not a security certification. AI approval does not prove an application is secure.
- Passing a pipeline is not a guarantee. Scanners have coverage limits and may miss vulnerabilities.
- Compliance requires evidence. Organizations may need documented approvals, audit records, risk acceptance, and policy enforcement.
- Governance must be proportionate. A training project may need a simple checklist; a regulated production system may need formal approvals, restricted AI access, and documented risk assessments.

For your DevSecOps training, the takeaway is that AI should be treated as an assistant within a governed security process, not as a replacement for developer judgment, security tools, or organizational accountability.
