# DevSecOps: DAST vs SAST vs Dependency Scanning

These three security testing methods find different types of security risks in your application. In a DevSecOps pipeline, they complement each other rather than replace each other.

## 1. The basic difference

[SAST, DAST, and Everything in Between: Making Sense of AppSec Technologies | Blog | DerScanner](https://images.openai.com/static-rsc-4/e7d9kAAtlkTE9CpJ90tj2PVeT69UdxNG3isEK2nh6z8a07oeImHgPKvTatMGU3O_XaPl8OlunzDLuUNw97Vr18SrJYyXuWpGdJvaT5h4jw9JyRBzJBJ3kzXSzZmBarrpKL1DGR62fr7m2984CDfnHFTH_V6wRReQAabONOl5Mls?purpose=inline)

SAST — Static Application Security Testing

Examines your source code without running the application to identify potentially insecure coding patterns.

Example: Detecting unsafe handling of user input in C# or TypeScript.

[What is SCA? Software Composition Analysis (2026)](https://images.openai.com/static-rsc-4/aZIpLpT7n4MKQ8T4FROqIqJg7jRQ_GUPRor4sic1s9deQSuHYf4v5Nwbuh3nW5UJ41vSsop35m-O7uIzkllzepcwdDe9UOXdK-wIgxOdDTHjP7rTlgz04dO9i77_6AMaCoseJdLifMnqGJGqYqOl1OZt-K0oD3dSrnqDSMkbZOk?purpose=inline)

Dependency scanning — SCA

Examines third-party libraries and packages your application depends on, looking for known vulnerabilities.

Example: Finding a vulnerable NuGet package or npm dependency associated with a CVE.

[SAST vs DAST vs IAST vs RASP: Complete Guide 2025](https://images.openai.com/static-rsc-4/7bO_hZCbs6EV1-9LUcSaw5AJ-BhkwAiIZl29Y3QReDuj7hDckrRCkPCBVCpNJw_hzOulotuInh_WFq3cOzPwsL1EgSVDLw1hfQpil-C7A-sQXPwaRYCd8jdFQiC3ew4GLXtp4iKHSXJwComv_NkfhTbjAsl23VN3k3WMavu4fEA?purpose=inline)

DAST — Dynamic Application Security Testing

Tests a running application by sending requests and analyzing responses for security weaknesses.

Example: Detecting missing security headers or testing whether an input is vulnerable to cross-site scripting.

## 2. Compare them side by side

| Feature               | SAST                             | Dependency scanning (SCA)                      | DAST                                          |
| --------------------- | -------------------------------- | ---------------------------------------------- | --------------------------------------------- |
| What it scans         | Source code                      | Third-party packages                           | Running application                           |
| Application must run? | No                               | No                                             | Yes                                           |
| Typical findings      | Unsafe code patterns             | Known library vulnerabilities                  | Runtime vulnerabilities and misconfigurations |
| CVE detection         | Sometimes, depending on the tool | Yes, for supported dependencies and advisories | Not its primary purpose                       |
| Typical tools         | GitLab SAST, Semgrep             | NuGet audit, npm audit, Trivy                  | OWASP ZAP, Burp Suite                         |
| Best time to run      | During code changes              | During dependency updates and builds           | After deployment to a test environment        |

Important: Trivy image scanning and dependency scanning overlap, but they are not identical. Trivy can inspect operating-system packages and application libraries inside a container image, while package-manager scans such as `npm audit` and NuGet auditing focus on dependency data available to those tools.

## 3. Practical example using your .NET and Angular project

Suppose your application has these three problems:

SAST

Problem: Potentially unsafe HTML handling

```
element.innerHTML = userInput;
```

A static analyzer can flag this pattern without starting Angular.

Dependency scanning

Problem: A vulnerable NuGet package

Your project uses a package version affected by a known security advisory. A dependency scanner compares the package and version with vulnerability data and reports the issue.

DAST

Problem: Missing HTTP security headers

You start your NGINX web server and scan it with ZAP. The scanner reports missing headers, such as `X-Content-Type-Options` or a suitable Content Security Policy.

Each tool catches a different class of problem. A SAST finding is not necessarily exploitable, a vulnerable dependency is not necessarily reachable in your application, and a DAST scan can miss vulnerabilities it does not exercise.

## 4. Where do they fit in a CI/CD pipeline?

Developer commits code

SAST

Analyze source

SCA

Check packages

Image scan

Scan container

Build and deploy to a test environment

DAST — Test the running application

This is a typical workflow; teams may run some checks in parallel or at different stages.

## 5. Remember it this way

- SAST: Is there potentially insecure code?
- SCA: Are my dependencies vulnerable?
- DAST: Does my running application expose security weaknesses?
- Trivy image scan: Does my container image contain known vulnerabilities or other detected security issues?

For your DevSecOps project, you've already experimented with dependency scanning, Trivy, and ZAP against NGINX. The next useful step is to run ZAP against your own running .NET/Angular application so you can compare findings from the same project across all three testing approaches.
