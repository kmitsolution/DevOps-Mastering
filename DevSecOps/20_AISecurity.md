# For your DevSecOps project in GitLab, AI-assisted security recommendations and test generation can help developers find potential security weaknesses, suggest safer code, and create tests for security-sensitive behavior.

The exact AI features depend on your GitLab edition, GitLab version, and GitLab Duo entitlement. Treat AI output as a recommendation that must be reviewed and validated—not as proof that code is secure.

1. AI-assisted security recommendations
Suppose your .NET 10 API accepts a user ID and returns user information.

Potentially risky example:
```
app.MapGet("/user", (string id) =>
{
    var query = $"SELECT * FROM Users WHERE Id = '{id}'";
    // Execute query
    return Results.Ok(query);
});
```

The query construction illustrates a potential SQL injection risk if it is executed against a database.

Ask GitLab Duo Code Suggestions or another AI assistant enabled in your environment:
Review this ASP.NET Core endpoint for security vulnerabilities. Focus on SQL injection, input validation, authentication and authorization, sensitive data exposure, and error handling. Recommend secure changes and explain the risks. Do not assume that authentication or database protections exist unless shown in the code.




A good recommendation would be to use parameterized queries or Entity Framework Core parameterization instead of concatenating untrusted input into SQL. The AI should also identify missing context rather than assume the endpoint is fully protected.

Other security areas you can ask it to review include:
- .NET API: authentication, authorization, input validation, secrets, and error handling.
- Angular UI: unsafe HTML rendering, XSS risks, and client-side validation limitations.
- Docker: running as root, unnecessary packages, exposed ports, and image selection.
- GitLab CI/CD: exposed credentials, unsafe variables, and overly permissive job permissions.
2. AI-generated security tests
For your project, you can ask GitLab Duo to generate tests for your ASP.NET Core API and Angular application. GitLab documents test generation through Duo Chat and IDE integrations, subject to feature access. 


GitLab Docs
+1


```
Example A: .NET API tests
Select the endpoint or service code in your IDE and ask:
Generate MSTest unit tests for this ASP.NET Core API.
```
Include:
1. Valid input and expected success response.
2. Missing, empty, and invalid input.
3. Unauthorized access, where authentication is required.
4. Forbidden access for users without permission.
5. Unexpected errors and safe error responses.
Use the existing project conventions. Do not invent application behavior. Identify which cases require integration tests or authentication setup, and explain how to run the tests.




Review the generated tests, then run them locally:
dotnet test DevSecOpsDemo.csproj


If the tests live in a separate test project, run dotnet test against that test project's .csproj instead.
Example B: Angular security tests
For your Angular UI, select the component or service and request:
Generate Angular unit tests using Jasmine and Karma for this component.

Test:
1. Valid and invalid user input.
2. Empty values and boundary conditions.
3. Error responses from the API.
4. Safe rendering of untrusted user input.
5. That client-side validation is not treated as a replacement for server-side validation.
Use the existing component and service interfaces. Include any necessary mocks and explain how to run the tests.




Run the tests using the scripts configured in angular-ui/package.json, commonly:
cd angular-ui
npm ci
npm test -- --watch=false


The exact command depends on your Angular test-runner configuration.
3. How this fits into your DevSecOps pipeline

1. AI assistance during development

Generate code, security recommendations, and candidate tests.




2. Merge request review

Review the diff and AI suggestions; request changes where needed.




3. Automated security checks

Run unit tests, SAST, dependency scanning, and Trivy image scanning.




4. Enforce pipeline results

Fail the pipeline when required tests or security policies fail.



AI-generated tests do not automatically run in CI/CD just because they were generated. Commit the reviewed tests and configure the appropriate test jobs in .gitlab-ci.yml.
4. What is available in GitLab?
Capability	Availability note
Code suggestions	GitLab Duo entitlement and IDE setup required; some GitLab.com Free usage may be available through GitLab Credits.  GitLab Docs +1
Test generation	Available through supported Duo features and IDE workflows, depending on subscription.  GitLab Docs +1
AI explanation and remediation of SAST findings	Advanced vulnerability explanation and resolution features have specific Ultimate and Duo add-on requirements.  GitLab Docs +1
Automated tests and scanners in CI/CD	Configure these separately; they provide repeatable validation regardless of whether AI was used to write the tests.

Important: AI-generated code and tests can be incorrect or incomplete. Review them, run the tests, and retain conventional security scanning. 


GitLab Docs
+1



For your hands-on lab, the next step is to confirm your GitLab setup so the instructions match the features you can actually use.

Are you using GitLab.com Free, or GitLab Self-Managed (your own GitLab server)?
