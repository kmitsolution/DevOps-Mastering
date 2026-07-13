For an **ASP.NET Core + MSTest** project, you should not commit build outputs, Visual Studio files, NuGet packages, or test result files to GitLab.

Create a file named:

```text
.gitignore
```

and add the following content:

```gitignore
## Ignore Visual Studio temporary files
.vs/
*.user
*.suo
*.userosscache
*.sln.docstates

## Build folders
bin/
obj/

## Publish folders
publish/

## Rider
.idea/

## VS Code
.vscode/

## NuGet Packages
*.nupkg
packages/

## Test Results
TestResults/
*.trx
*.coverage
*.coveragexml

## JUnit Reports
*.xml

## Logs
*.log

## ASP.NET Scaffolding
ScaffoldingReadMe.txt

## IIS Express
IISExpress/

## Azure
PublishProfiles/

## Resharper
_ReSharper*/
*.DotSettings.user

## SonarQube
.sonarqube/

## Docker
docker-compose.override.yml

## OS files
Thumbs.db
Desktop.ini
.DS_Store
```

---

# Important Note About `*.xml`

Be careful with:

```gitignore
*.xml
```

This ignores **all XML files**, including:

* `app.config`
* `web.config`
* `NuGet.config`

which you may actually want in Git.

A better approach is:

```gitignore
TestResults/
*.trx
```

or:

```gitignore
reports/
```

instead of ignoring all XML files.

---

# Recommended `.gitignore`

This is what I recommend for your GitLab CI course:

```gitignore
## Visual Studio
.vs/
*.user
*.suo

## Build Output
bin/
obj/
publish/

## Test Results
TestResults/
*.trx

## VS Code
.vscode/

## Rider
.idea/

## Logs
*.log

## ReSharper
_ReSharper*/

## SonarQube
.sonarqube/

## OS Files
Thumbs.db
.DS_Store
Desktop.ini
```

---

# Why Ignore These Files?

| Folder/File    | Reason                         |
| -------------- | ------------------------------ |
| `.vs/`         | Visual Studio local settings   |
| `bin/`         | Compiled DLLs and executables  |
| `obj/`         | Intermediate build files       |
| `publish/`     | Generated during CI/CD         |
| `TestResults/` | Generated during unit testing  |
| `*.trx`        | Test report files              |
| `.vscode/`     | User-specific VS Code settings |
| `.idea/`       | Rider IDE settings             |

---

# Example Repository Structure After `.gitignore`

Files that **will be pushed**:

```text
ProductApi.sln
ProductApi/
ProductUnitTest/
.gitlab-ci.yml
.gitignore
README.md
```

Files that **will not be pushed**:

```text
bin/
obj/
publish/
.vs/
TestResults/
*.trx
```

---

# Verify Before Pushing

Run:

```bash
git status
```

You should see only source files:

```text
new file: ProductApi/Program.cs
new file: ProductUnitTest/Test1.cs
new file: ProductApi.sln
new file: .gitlab-ci.yml
new file: .gitignore
```

You should **not** see:

```text
bin/
obj/
.vs/
TestResults/
```

If you already committed them earlier, remove them from Git tracking:

```bash
git rm -r --cached bin obj .vs TestResults
git commit -m "Added .gitignore"
```

This is the standard `.gitignore` typically used for ASP.NET Core projects in GitLab and GitHub repositories.
