
## Step 1 — Update `.dockerignore`

In your project root, open `.dockerignore` and make sure it contains:

```

bin/
obj/
.git/
.vs/
.vscode/
TestResults/
coverage/
angular-ui/node_modules/
angular-ui/dist/

```

The important entries are `bin/` and `obj/`. They prevent Windows-generated build assets from being copied into the Linux container.

## Step 2 — Use this Dockerfile

Replace your current Dockerfile with the following. It copies the project and lock file first, restores dependencies inside Linux, and then copies the application source.

```

FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build

WORKDIR /src

# Copy project and NuGet lock file
COPY DevSecOpsDemo.csproj packages.lock.json ./

# Restore packages inside the Linux container
RUN dotnet restore DevSecOpsDemo.csproj

# Copy source code; .dockerignore excludes bin and obj
COPY . .

# Publish using the Linux-generated restore assets
RUN dotnet publish DevSecOpsDemo.csproj \
    --configuration Release \
    --output /app/publish \
    --no-restore

FROM mcr.microsoft.com/dotnet/aspnet:10.0 AS final

WORKDIR /app

ENV ASPNETCORE_HTTP_PORTS=8080

EXPOSE 8080

COPY --from=build /app/publish .

ENTRYPOINT ["dotnet", "DevSecOpsDemo.dll"]

```

## Step 3 — Rebuild from the correct folder

In PowerShell, make sure you're in the folder containing the Dockerfile, `.dockerignore`, `DevSecOpsDemo.csproj`, and `packages.lock.json`.

Run:

```
docker build --no-cache -t devsecops-demo:1.0 .
```

Check the build output: it should show a successful `dotnet restore` before `COPY . .`, and then a successful publish. Your previous log showed only six build steps, with publish immediately after `COPY . .`, suggesting Docker was still using the old Dockerfile or the updated changes had not been saved.

If it still fails, run this command to find where the fallback path is configured:

```
Get-ChildItem Env:NUGET*
dotnet nuget locals all --list
Get-ChildItem "$env:APPDATA\NuGet" -Filter "NuGet.Config" -Recurse -ErrorAction SilentlyContinue
```

Also search for the path in project configuration files and generated assets:

```
Get-ChildItem -Path . -Recurse -File -Include *.json,*.config,*.props,*.targets |
  Select-String -SimpleMatch 'C:\Program Files (x86)\Microsoft Visual Studio\Shared\NuGetPackages'
```

## Step 4 — Run the image after a successful build

```
docker run -d --name devsecops-demo -p 8080:8080 devsecops-demo:1.0
```

Then check:

```
docker ps
docker logs devsecops-demo
```

One final note: your `.csproj` includes `System.Text.Encodings.Web` version `10.0.0`. That is separate from the fallback-folder error. For your security training, we can review whether this explicit reference is necessary after the Docker build succeeds.
