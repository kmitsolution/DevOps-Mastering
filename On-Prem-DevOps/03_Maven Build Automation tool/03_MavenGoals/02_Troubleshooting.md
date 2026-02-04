
# Common Maven Build Failures (DevOps View)

In **Apache Maven**, most build failures fall into a few repeatable categories.

---

## 1️⃣ ❌ “The goal you specified requires a project to execute but there is no POM”

### Error

```text
MissingProjectException
There is no POM in this directory
```

### Cause

* Running Maven from the wrong directory
* `pom.xml` not present

### Fix

```bash
cd <project-root>
ls pom.xml
mvn clean package
```

📌 **Rule:** Maven commands (except archetype/help/version) must be run **where pom.xml exists**

---

## 2️⃣ ❌ Dependency Could Not Be Resolved

### Error

```text
Could not resolve dependencies for project
Could not find artifact
```

### Causes

* No internet access
* Wrong dependency version
* Nexus/Artifactory down
* Proxy not configured

### Fix

```bash
mvn clean install -U
```

Check:

```text
~/.m2/settings.xml
```

---

## 3️⃣ ❌ Tests Failing (Surefire Failure)

### Error

```text
There are test failures
maven-surefire-plugin:test failed
```

### Causes

* Unit test failure
* Environment-specific tests
* Missing test configuration

### Fix (Temporary – DevOps)

```bash
mvn clean package -DskipTests
```

⚠️ **Don’t skip tests permanently in CI**

---

## 4️⃣ ❌ Java Version Mismatch

### Error

```text
invalid target release
Unsupported class file major version
```

### Causes

* Java version on server ≠ pom.xml version
* Wrong JDK configured in CI

### Fix

Check Java:

```bash
java -version
```

Fix pom.xml:

```xml
<maven.compiler.source>21</maven.compiler.source>
<maven.compiler.target>21</maven.compiler.target>
```

---

## 5️⃣ ❌ Plugin Not Found / Plugin Version Issue

### Error

```text
Plugin could not be resolved
```

### Causes

* Plugin version not specified
* Repository unavailable

### Fix

Always specify plugin versions:

```xml
<plugin>
  <artifactId>maven-compiler-plugin</artifactId>
  <version>3.11.0</version>
</plugin>
```

---

## 6️⃣ ❌ Corrupted Local Repository

### Error

```text
zip END header not found
Could not read artifact descriptor
```

### Cause

* Interrupted dependency download
* Corrupt `.m2` cache

### Fix

Delete and rebuild:

```bash
rm -rf ~/.m2/repository
mvn clean install
```

---

## 7️⃣ ❌ Proxy / Network Issues (Very Common in Enterprises)

### Error

```text
Connection timed out
Could not transfer artifact
```

### Cause

* Corporate proxy
* Firewall restrictions

### Fix (`settings.xml`)

```xml
<proxies>
  <proxy>
    <host>proxy.company.com</host>
    <port>8080</port>
  </proxy>
</proxies>
```

---

## 8️⃣ ❌ Build Works Locally but Fails in CI

### Causes

* Different Java versions
* Missing environment variables
* Cached dependencies locally

### Fix

Align versions:

```text
Local Java = CI Java
Local Maven = CI Maven
```

Use:

```bash
mvn -B clean package
```

---

## 9️⃣ ❌ SNAPSHOT Dependency Issues

### Error

```text
Could not find SNAPSHOT artifact
```

### Causes

* Snapshot not deployed
* CI using stale cache

### Fix

```bash
mvn clean install -U
```

---

## 🔟 ❌ Permission Issues

### Error

```text
Permission denied
Access denied
```

### Causes

* No write access to:

  * `target/`
  * `.m2/`
* CI agent permissions

### Fix

```bash
chmod -R 755 project-dir
```

---

## 🔍 Debugging Maven Failures (Must Know)

| Option | Purpose         |
| ------ | --------------- |
| `-X`   | Full debug logs |
| `-e`   | Stack trace     |
| `-U`   | Force update    |
| `-B`   | Batch mode      |

Example:

```bash
mvn clean package -X -e
```

---

## 🧠 DevOps Golden Rules ⭐

1. Always specify **plugin versions**
2. Align **Java + Maven versions**
3. Cache `.m2` in CI
4. Use `-B` in pipelines
5. Never hardcode credentials in `pom.xml`

---

## Interview One-Liners ⭐

> **Most Maven build failures are caused by dependency resolution issues, Java version mismatches, or environment differences between local and CI systems.**

> **Running Maven with `-X` and `-e` helps debug build failures effectively.**

---

## Quick Memory Cheat Sheet 🧠

```text
No POM           → wrong directory
Dependency error → repo / proxy / version
Test failure     → surefire
Java error       → JDK mismatch
CI failure       → environment mismatch
```

