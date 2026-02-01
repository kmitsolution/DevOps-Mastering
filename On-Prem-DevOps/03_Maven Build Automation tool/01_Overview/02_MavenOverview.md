## Apache Maven

**Apache Maven** is a **build automation and project management tool**.

In simple words 👉

> Maven **automates building, testing, packaging, and deploying applications**.

For DevOps engineers, Maven is mainly used to:

* Build applications automatically
* Manage dependencies
* Produce consistent artifacts for CI/CD pipelines

---

## Who Built Maven?

Maven is developed and maintained by the **Apache Software Foundation**.

* Open-source
* Community-driven
* Widely used in enterprise environments

---

## Why Maven Was Created (Quick Context)

Before Maven:

* Builds were manual or Ant-based
* No standard project structure
* Dependency management was painful

Maven introduced:

* **Convention over configuration**
* Standard directory layout
* Centralized dependency management

---

## Maven Build Lifecycle (Very Important)

A **lifecycle** is a **sequence of phases** Maven follows to build a project.

Maven has **3 main lifecycles**:

1. Default
2. Clean
3. Site

---

## Default Lifecycle Phases (Text Diagram)

This is the lifecycle **used in CI/CD pipelines** 

```text
validate
   ↓
compile
   ↓
test
   ↓
package
   ↓
verify
   ↓
install
   ↓
deploy
```

### What Each Phase Does (Short & Practical)

* **validate** → checks project structure
* **compile** → compiles source code
* **test** → runs unit tests
* **package** → creates JAR/WAR file
* **verify** → verifies quality checks
* **install** → installs artifact to local repo (.m2)
* **deploy** → pushes artifact to remote repo (Nexus/Artifactory)

--> If you run:

```bash
mvn package
```

Maven automatically runs:

```text
validate → compile → test → package
```

---

## Clean Lifecycle (Text Diagram)

Used to **clean old builds** 

```text
clean
```

* Deletes the `target/` directory
* Common in pipelines:

```bash
mvn clean package
```

---

## Site Lifecycle (Text Diagram)

Used for **project documentation** 

```text
site
   ↓
site-deploy
```

Mostly used for reports, not common in CI/CD.

---

## Is Maven Only for Java?

### Short Answer ❌ No

### Practical Answer ✅ Mostly used with Java

Maven is **language-agnostic**, but:

* It is **primarily designed for Java ecosystem**
* Most plugins are Java-focused

### Commonly Used With:

* **Java**
* Scala
* Kotlin

### Can Maven Be Used for Other Languages?

Yes, but **not ideal**:

* Python → better with pip / poetry
* Node.js → npm / yarn
* .NET → MSBuild

 That’s why in real DevOps projects:

* Java → Maven / Gradle
* Node.js → npm
* Python → PyBuilder / Poetry
* .NET → MSBuild

---

## Maven in CI/CD (DevOps View)

```text
Git Commit
   ↓
CI Tool (Jenkins)
   ↓
mvn clean test package
   ↓
Artifact (JAR/WAR)
   ↓
Nexus / Artifactory
   ↓
Deploy
```

---

## Interview-Ready One-Line Summary ⭐

> **Maven is an Apache build automation tool mainly used for Java projects to manage dependencies, build lifecycle, and create deployable artifacts in CI/CD pipelines.**

---


