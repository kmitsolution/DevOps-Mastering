# Understanding `pom.xml` (Maven Project Point of View)

In **Apache Maven**, `pom.xml` stands for **Project Object Model**.

👉 **`pom.xml` is the blueprint of a Maven project**
It tells Maven:

* What to build
* How to build
* With which dependencies
* With which plugins
* What artifact to produce

---

## 1️⃣ `groupId`, `artifactId`, `version` (Project Identity)

These three together **uniquely identify your project**.

```xml
<groupId>com.kmitcourses</groupId>
<artifactId>hello-world</artifactId>
<version>1.0-SNAPSHOT</version>
```

### What They Mean

| Element      | Meaning                                 |
| ------------ | --------------------------------------- |
| `groupId`    | Organization / company (reverse domain) |
| `artifactId` | Project name                            |
| `version`    | Project version                         |

### Example Artifact Name

```text
hello-world-1.0-SNAPSHOT.jar
```

### DevOps Importance ⭐

* Used by **Nexus / Artifactory**
* Used for versioning & rollback
* Required for CI/CD artifact promotion

---

## 2️⃣ Dependencies (What Your App Needs)

Dependencies are **external libraries** your project requires.

```xml
<dependencies>
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
  </dependency>
</dependencies>
```

### Key Concepts

* Maven **downloads dependencies automatically**
* Stored in:

```text
~/.m2/repository
```

### Dependency Scope (Important)

| Scope      | Used When                   |
| ---------- | --------------------------- |
| `compile`  | Needed at compile & runtime |
| `test`     | Only for testing            |
| `provided` | Provided by server (Tomcat) |
| `runtime`  | Only at runtime             |

### DevOps View

* CI servers download dependencies
* `.m2` cache improves pipeline speed
* Dependency conflicts cause build failures

---

## 3️⃣ Properties (Centralized Configuration)

Properties help avoid **hardcoding values**.

```xml
<properties>
  <java.version>21</java.version>
  <maven.compiler.source>21</maven.compiler.source>
  <maven.compiler.target>21</maven.compiler.target>
</properties>
```

### Why Properties Matter

* Single place to change values
* Easy Java upgrade
* CI/CD friendly

---

## 4️⃣ Build Section (How the Project Is Built)

The `<build>` section defines **build behavior**.

```xml
<build>
  <finalName>hello-world</finalName>
</build>
```

* `finalName` controls artifact name
* Default output directory → `target/`

---

## 5️⃣ Plugins (Build Automation Power)

Plugins perform **actual build work**.

### Maven Compiler Plugin (Very Important)

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.11.0</version>
      <configuration>
        <source>${maven.compiler.source}</source>
        <target>${maven.compiler.target}</target>
      </configuration>
    </plugin>
  </plugins>
</build>
```

### What This Plugin Does

* Compiles Java code
* Controls Java version

### DevOps Tip ⭐

Mismatch between:

```text
Java version on server
Java version in pom.xml
```

→ **Build failure**

---

## 6️⃣ Plugin vs Dependency (Very Common Confusion)

| Dependency                   | Plugin                |
| ---------------------------- | --------------------- |
| Used by application code     | Used by Maven         |
| Goes inside `<dependencies>` | Goes inside `<build>` |
| Example: Spring, JUnit       | Compiler, Surefire    |

---

## 7️⃣ Build Lifecycle + Plugins (How It Works Together)

```text
mvn clean package
   ↓
Compiler plugin → compile
Surefire plugin → test
Jar plugin → package
```

Plugins are **bound to lifecycle phases**.

---

## 8️⃣ Example `pom.xml` (DevOps-Ready)

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.kmitcourses</groupId>
  <artifactId>hello-world</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>jar</packaging>

  <properties>
    <maven.compiler.source>21</maven.compiler.source>
    <maven.compiler.target>21</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.13.2</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.11.0</version>
      </plugin>
    </plugins>
  </build>

</project>
```

---

## 9️⃣ CI/CD Perspective Summary ⭐

| pom.xml Section              | Why DevOps Cares        |
| ---------------------------- | ----------------------- |
| `groupId/artifactId/version` | Artifact identity       |
| `dependencies`               | Build reliability       |
| `properties`                 | Environment consistency |
| `plugins`                    | Automation              |
| `build`                      | Repeatable builds       |

---

## Interview One-Liner ⭐

> **`pom.xml` defines the project identity, dependencies, build lifecycle, and plugins required to build and package a Maven project consistently across environments.**

---

