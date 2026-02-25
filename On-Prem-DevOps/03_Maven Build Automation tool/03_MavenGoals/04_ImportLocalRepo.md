Perfect 👍 — now I’ll give you a **complete, real-world, step-by-step tutorial** from scratch:

1️⃣ Create Maven Quickstart Project
2️⃣ Add custom method
3️⃣ Install it into local repository
4️⃣ Create second project
5️⃣ Import dependency
6️⃣ Run correctly

This is exactly how Maven is used in real DevOps workflows.

---

# 🔹 PART 1 — Create First Maven Project (Library Project)

We will create a reusable JAR.

---

## Step 1: Generate Maven Quickstart

```bash
mvn archetype:generate -B \
-DgroupId=com.kmitcourses \
-DartifactId=basic_mvnproj \
-DarchetypeArtifactId=maven-archetype-quickstart \
-DarchetypeVersion=1.4
```

Now you will see:

```
basic_mvnproj/
 ├── pom.xml
 └── src/
```

---

## Step 2: Modify App.java

Go to:

```
basic_mvnproj/src/main/java/com/kmitcourses/App.java
```

Update it like this:

```java
package com.kmitcourses;

public class App {

    public String getMessage() {
        return "Hello from KMITS Courses Library!";
    }
}
```

(We don’t need main method here — this is a library project.)

---

## Step 3: Build and Install to Local Repository

From inside `basic_mvnproj` folder:

```bash
mvn clean install
```

✔ This creates:

```
~/.m2/repository/com/kmitcourses/basic_mvnproj/1.0-SNAPSHOT/
```

Inside it:

```
basic_mvnproj-1.0-SNAPSHOT.jar
```

Now your library is ready for reuse.

---

# 🔹 PART 2 — Create Second Maven Project (Consumer Project)

Now we will use that library.

---

## Step 4: Create Second Project

```bash
mvn archetype:generate -B \
-DgroupId=com.example \
-DartifactId=demo \
-DarchetypeArtifactId=maven-archetype-quickstart \
-DarchetypeVersion=1.4
```

Now go inside:

```
demo/
```

---

## Step 5: Add Dependency in demo/pom.xml

Open `demo/pom.xml` and add:

```xml
<dependency>
    <groupId>com.kmitcourses</groupId>
    <artifactId>basic_mvnproj</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

Full dependency section example:

```xml
<dependencies>
    <dependency>
        <groupId>com.kmitcourses</groupId>
        <artifactId>basic_mvnproj</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>
```

---

## Step 6: Use the Library Class

Go to:

```
demo/src/main/java/com/example/App.java
```

Replace with:

```java
package com.example;

import com.kmitcourses.App;

public class App {
    public static void main(String[] args) {
        System.out.println("Demo Project Started");

        App libApp = new App(); // ❌ This will conflict name
    }
}
```

⚠ We must avoid class name conflict.

So better rename local class to:

```
DemoApp.java
```

Final correct version:

```java
package com.example;

import com.kmitcourses.App;

public class DemoApp {

    public static void main(String[] args) {

        System.out.println("Demo Project Started");

        App library = new App();
        System.out.println(library.getMessage());
    }
}
```

Delete old App.java file if needed.

---

## Step 7: Compile

```bash
mvn clean compile
```

✔ Should succeed.

---

# 🔹 PART 3 — Run the Application

---

## Option 1 (Manual Classpath — What You Learned)

```bash
java -cp "target/demo-1.0-SNAPSHOT.jar;C:\Users\<user>\.m2\repository\com\kmitcourses\basic_mvnproj\1.0-SNAPSHOT\basic_mvnproj-1.0-SNAPSHOT.jar" com.example.DemoApp
```

---

## Option 2 (Recommended — Maven Exec Plugin)

Add this inside `<build>`:

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.codehaus.mojo</groupId>
      <artifactId>exec-maven-plugin</artifactId>
      <version>3.1.0</version>
    </plugin>
  </plugins>
</build>
```

Then run:

```bash
mvn exec:java -Dexec.mainClass="com.example.DemoApp"
```

✔ Dependencies handled automatically.

---

## Option 3 (Best Practice — Create Fat Jar)

Use `maven-shade-plugin` to bundle dependencies (recommended for CLI apps).

---

# 🔹 What You Just Achieved

You implemented:

```
Project 1 → mvn install → Local Repository
Project 2 → Dependency → Reuse Library
```

This is the foundation of:

* Multi-module projects
* Enterprise builds
* CI/CD artifact reuse
* Nexus / Artifactory usage

---

# 🔹 Important Concepts Learned

| Concept              | Meaning                      |
| -------------------- | ---------------------------- |
| mvn install          | Installs jar to local repo   |
| groupId + artifactId | Unique project identity      |
| Dependency           | Reuses another project       |
| Compile vs Runtime   | Different classpath behavior |

---

# 🔥 Real DevOps Upgrade (Better Approach)

Instead of using `mvn install`, in enterprise we use:

```
mvn deploy
```

→ Upload to Nexus / Artifactory
→ Other teams download it

---

# 🎯 Interview-Level Summary

> A Maven project can be reused by installing it into the local repository using `mvn install`, then adding it as a dependency in another project's `pom.xml`.

---
