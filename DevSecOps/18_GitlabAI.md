# GitLab AI — Overview for DevOps and DevSecOps

GitLab AI refers to AI-powered capabilities integrated into the GitLab software development platform. These capabilities help developers, DevOps engineers, security teams, and platform engineers automate routine work, understand code, troubleshoot pipelines, and improve software delivery.

GitLab's AI capabilities have included GitLab Duo, AI-assisted code suggestions, chat, code explanations, and AI support for security and CI/CD workflows. The exact capabilities available depend on the GitLab version, subscription, and enabled AI features.

## 1. Where GitLab AI fits in the DevOps lifecycle

Plan

Help understand issues and create implementation plans

Code

Generate suggestions, explain code, and help write tests

Build and CI/CD

Help explain pipeline configuration and job failures

Secure

Help interpret vulnerability findings and recommend remediation

Review and maintain

Summarize changes and assist with code reviews and troubleshooting

## 2. Important GitLab AI capabilities

GitLab Duo Chat and Agentic Chat

Ask questions about code, GitLab workflows, and project tasks. Agentic capabilities can assist with more complex, multi-step work, depending on the enabled feature and permissions.

Code Suggestions

Suggests code while developers work in supported IDEs. It can help generate code, complete functions, and speed up repetitive coding tasks.

Merge request assistance

AI features can help summarize code changes, explain modifications, and support review workflows when available in the configured GitLab offering.

CI/CD assistance

AI can help explain `.gitlab-ci.yml`, troubleshoot failed jobs, and suggest pipeline improvements. Some agentic workflows can assist with debugging and making changes.

Security assistance

Depending on the plan and enabled features, AI can help explain security findings, identify potential remediation steps, and assist with vulnerability resolution.

GitLab Duo's feature set evolves, and not every capability is included in every subscription. The official documentation describes the current capabilities and requirements.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.gitlab.com\&sz=32)

GitLab Docs

+1



## 3. Practical example for your DevSecOps project

Suppose your GitLab pipeline fails with this error:

```
ERROR: Job failed: exit code 1
```

You can use an AI assistant to investigate systematically:

1. Ask it to explain the failing job and relevant `.gitlab-ci.yml` section.
2. Provide the full error log and relevant configuration.
3. Ask for the likely root cause and a minimal fix.
4. Apply the change in a feature branch.
5. Run the pipeline again and verify the result.

For example, a useful prompt is:

Analyze this GitLab CI/CD pipeline failure. Explain the root cause, identify the exact failing command, and suggest the smallest safe fix. Preserve the existing pipeline stages and security checks. Do not weaken security controls just to make the pipeline pass.

[Paste the failed job log and relevant .gitlab-ci.yml section here.]

AI-generated suggestions should still be reviewed and tested before merging.

## 4. Can GitLab AI run on-premises?

Yes, but there are different deployment options.

| Option                                    | How it works                                                                                                                                    |
| ----------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| GitLab Self-Managed with GitLab-hosted AI | Your GitLab instance connects to GitLab's hosted AI infrastructure. Internet connectivity and appropriate licensing/configuration are required. |
| GitLab Duo Self-Hosted                    | Your organization hosts the AI Gateway and supported language models, giving you more control over AI requests and responses.                   |
| Hybrid setup                              | Some AI features use your own infrastructure while others use GitLab-managed AI services.                                                       |

GitLab documents self-hosted and hybrid configurations, including options for environments that cannot make external AI calls. Licensing and supported features vary by product and version.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.gitlab.com\&sz=32)

GitLab Docs

+1



For a fully isolated environment, GitLab Duo Self-Hosted is the relevant architecture to investigate.

## 5. Suggested hands-on classroom exercise

Use your existing `.NET 10` and Angular project:

- Ask AI to explain your `.gitlab-ci.yml`.
- Ask it to generate a simple unit test for a .NET API endpoint.
- Ask it to explain a Trivy vulnerability finding.
- Ask it to troubleshoot a deliberately failing pipeline in a test branch.
- Review the proposed changes and compare them with the actual pipeline results.

This demonstrates AI assistance across coding, testing, CI/CD, and DevSecOps without treating AI output as automatically correct.

One question: Are you interested in using GitLab AI as a developer through VS Code, or in installing and configuring GitLab AI on an on-premises GitLab server? I can give you the practical steps for either setup.
