> **Website domain:** `kmitcourses.com`
> **GroupId:** `com.kmitcourses`

--

# Maven Archetype (Using `com.kmitcourses` Example)

## What is Archetype in Maven?

In **Apache Maven**, an **archetype** is a **project template**.

👉 **Archetype = Blueprint for a Maven project**

It automatically creates:

* Standard directory structure
* Sample source code
* `pom.xml`

---

## Why Use Domain-Based `groupId`?

Maven follows **reverse domain naming**:

```text
Domain: kmitcourses.com
groupId: com.kmitcourses
```

Benefits:

* Globally unique
* Enterprise standard
* Easy dependency identification

---

# Creating a Maven Project (HelloWorld)

## Step 1: Generate Project Using Archetype

```bash
mvn archetype:generate
```

---

## Step 2: Select Archetype

From the list:

```text
2: maven-archetype-quickstart which is by default selected so press enter key
```

# Step 3:

Choose org.apache.maven.archetypes:maven-archetype-quickstart version



select the latest version or press enter by default it has chosen the latest version

## Step 4: Enter Project Details

<img width="282" height="65" alt="image" src="https://github.com/user-attachments/assets/5b18e0d1-387f-4c43-8c04-a8acc8822790" />

```text
Define value for groupId:
com.kmitcourses

Define value for artifactId:
hello-world

Define value for version:
1.0-SNAPSHOT

Define value for package:
com.kmitcourses
```

Confirm:

```text
Y
```

---

## Meaning of Each Value

### groupId

```text
com.kmitcourses
```

* Company / organization identifier

---

### artifactId

```text
hello-world
```

* Project name
* JAR name

---

### version

```text
1.0-SNAPSHOT
```

* Snapshot = under development

---

### package

```text
com.kmitcourses.helloworld
```

* Java package name

---

# Generated Project Structure

```text
hello-world/
 ├── pom.xml
 └── src
     ├── main
     │   └── java
     │       └── com/kmitcourses
     │           └── App.java
     └── test
         └── java
             └── com/kmitcourses/
                 └── AppTest.java
```

---

## HelloWorld Source Code

### `App.java`

```java
package com.kmitcourses;

public class App {
    public static void main(String[] args) {
        System.out.println("Hello World from KMITS Courses!");
    }
}
```

---

## pom.xml (Key Highlights)

```xml
<groupId>com.kmitcourses</groupId>
<artifactId>hello-world</artifactId>
<version>1.0-SNAPSHOT</version>
<packaging>jar</packaging>
```

---

## Build the Project

From the project directory:

```bash
mvn clean package
```

### Lifecycle executed:

```text
validate → compile → test → package
```

Artifact created:

```text
target/hello-world-1.0-SNAPSHOT.jar
```

---

## Run the Application

```bash
java -cp target/hello-world-1.0-SNAPSHOT.jar com.kmitcourses.App
```

Output:

```text
Hello World from KMITS Courses!
```

---

## CI/CD Friendly (Non-Interactive Mode)

```bash

mvn archetype:generate -DgroupId=com.kmitcourses -DartifactId=hello-world -DarchetypeArtifactId=maven-archetype-quickstart -Dversion=1.0 -DinteractiveMode=false


```

Perfect for:

* Jenkins pipelines
* GitHub Actions
* GitLab CI

---

## Interview One-Liners ⭐

> **Maven archetypes provide predefined project templates that follow best practices and standard directory structures.**

> **Using reverse domain names like `com.kmitcourses` ensures uniqueness of Maven artifacts.**

---

If you want next, I can:

* 🧩 Create **custom archetype for KMITS Courses**
* 🚀 Add **Jenkins pipeline for this project**
* 🐳 Dockerize this Maven HelloWorld app
* ❓ Prepare **Maven interview questions**

Just say 👍
