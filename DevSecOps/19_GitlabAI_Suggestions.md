# GitLab AI: Code Suggestions and Merge Request Review Assistance

Let's look at two practical GitLab AI capabilities you can demonstrate in your DevSecOps class:

1. AI Code Suggestions — help developers write code in their IDE.
2. Merge Request (MR) Review Assistance — help developers understand changes, identify potential issues, and improve code reviews.

These capabilities are associated with GitLab Duo and related AI features. Availability depends on your GitLab edition, subscription, version, IDE, and configuration.

## 1. AI Code Suggestions

Code Suggestions use the context around your code to recommend completions while you type. Depending on the supported IDE and feature, they can suggest anything from a line of code to a larger block.

### Practical example: .NET 10 API

Suppose your `Program.cs` contains a simple ASP.NET Core application. You want to create an endpoint that returns a welcome message.

You start typing a comment or method, such as:

```
// Create an endpoint that returns a welcome message
// for a user supplied through a query parameter.
```

An AI code assistant may suggest the endpoint implementation. You review the suggestion, accept or modify it, and then build and test the application.

For example, a suitable implementation could be:

```

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/welcome", (string? name) =>
{
    var safeName = string.IsNullOrWhiteSpace(name)
        ? "Developer"
        : name.Trim();

    return Results.Ok(new
    {
        Message = $"Welcome, {safeName}!"
    });
});

app.Run();

```

Test it after running your application:

```
curl "http://localhost:8080/welcome?name=DevOps"
```

Example response:

```
{
  "message": "Welcome, DevOps!"
}
```

Classroom takeaway: AI generates a suggestion; the developer remains responsible for checking correctness, security, coding standards, and tests.

## 2. Set up AI Code Suggestions in VS Code

GitLab supports Code Suggestions in VS Code through its GitLab extension and Duo configuration.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.gitlab.com\&sz=32)

GitLab Docs

+1



1. Install the official GitLab extension for VS Code.
2. Sign in to GitLab and configure GitLab Duo access for your account.
3. Open your `.NET` or Angular repository.
4. Start typing code or write a descriptive comment.
5. Review the suggested code and accept it only when it meets your requirements.

For example, in an Angular TypeScript file, type:

```
// Create a function that validates an email address
// and returns true if the format is valid.
```

If Code Suggestions is enabled and your setup supports the language, the IDE may suggest an implementation. Review and test it rather than assuming the generated validation is sufficient for security-sensitive use cases.

## 3. Merge Request review assistance

GitLab Duo can help review proposed changes and provide feedback about potential errors, coding standards, and other issues. The exact review workflow depends on the available Duo features and subscription.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.gitlab.com\&sz=32)

GitLab Docs

+1



### Practical classroom exercise

Suppose you change a .NET endpoint in a feature branch:

```
app.MapGet("/user", (string id) =>
{
    return Results.Ok($"User ID: {id}");
});
```

Create a Merge Request with this change. In GitLab:

1. Open Code → Merge requests.
2. Open your new Merge Request.
3. If your subscription and instance support it, assign GitLab Duo as a reviewer or request a review using the `@GitLabDuo` interaction.
4. Review the AI feedback and decide whether each comment is valid.
5. Make any required corrections, run tests, and update the Merge Request.

An AI reviewer might suggest input validation, better error handling, or a test for the endpoint. Such suggestions are examples of possible feedback, not a prediction of exactly what GitLab will report.

## 4. What depends on your GitLab edition?

| Capability                   | Availability consideration                                                                                                      |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Code Suggestions             | Available through supported GitLab Duo configurations; the precise entitlement depends on your GitLab version and subscription. |
| AI-generated MR descriptions | Requires an eligible offering and Duo feature entitlement.                                                                      |
| AI Merge Request review      | Depends on which review feature and AI add-on are enabled.                                                                      |
| Self-managed/on-premises use | Requires compatible GitLab configuration, licensing, and AI connectivity or supported self-hosted models.                       |

Current official documentation distinguishes basic Code Suggestions from more advanced AI review features.&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.gitlab.com\&sz=32)

GitLab Docs

+2



To make this practical for your class: are you using GitLab.com Free, or a paid GitLab Self-Managed instance? That determines which AI features your students can actually try.
