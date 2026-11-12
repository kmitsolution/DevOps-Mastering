
# Maven Commands (Goals & Phases Explained)

In **Apache Maven**, when you run a Maven command, you are actually running **goals** that belong to **plugins**, and those goals are executed during **lifecycle phases**.

---

## 1️⃣ Maven Lifecycle vs Phase vs Goal (First Understand This)

### Lifecycle

A **lifecycle** is a **sequence of phases**.

### Phase

A **phase** is a **step in the lifecycle**.

### Goal

A **goal** is an **action performed by a plugin**.

### Simple Mapping

```text
Lifecycle → Phase → Goal → Plugin
```

---

## 2️⃣ Main Maven Lifecycles

Maven has **3 built-in lifecycles**:

| Lifecycle | Purpose                      |
| --------- | ---------------------------- |
| default   | Build, test, package, deploy |
| clean     | Cleanup                      |
| site      | Documentation                |

---

## 3️⃣ Default Lifecycle Phases (Most Important)

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

👉 Running a **later phase automatically runs all earlier phases**.

---

## 4️⃣ Most Used Maven Commands (With Explanation)

### 🔹 `mvn validate`

```bash
mvn validate
```

✔ Checks project structure
✔ Validates `pom.xml`

Used in CI to fail fast.

---

### 🔹 `mvn compile`

```bash
mvn compile
```

✔ Compiles source code
✔ Downloads dependencies

Output:

```text
target/classes
```

---

### 🔹 `mvn test`

```bash
mvn test
```

✔ Runs unit tests
✔ Uses **Surefire plugin**

Output:

```text
target/surefire-reports
```

---

### 🔹 `mvn package`

```bash
mvn package
```

✔ Compiles
✔ Runs tests
✔ Creates artifact (`jar` / `war`)

Output:

```text
target/hello-world-1.0-SNAPSHOT.jar
```

---

### 🔹 `mvn verify`

```bash
mvn verify
```

✔ Runs integration tests
✔ Performs quality checks

---

### 🔹 `mvn install`

```bash
mvn install
```

✔ Builds project
✔ Installs artifact into **local repo**

```text
~/.m2/repository/com/kmitcourses/hello-world/
```

---

### 🔹 `mvn deploy`

```bash
mvn deploy
```

✔ Uploads artifact to **remote repo**
✔ Used in CI/CD pipelines

Examples:

* Nexus
* Artifactory

---

## 5️⃣ Clean Lifecycle Commands

### 🔹 `mvn clean`

```bash
mvn clean
```

✔ Deletes `target/` directory

Common usage:

```bash
mvn clean package
```

---

## 6️⃣ Site Lifecycle Commands

### 🔹 `mvn site`

```bash
mvn site
```

✔ Generates project documentation

Output:

```text
target/site
```

---

## 7️⃣ Maven Goals (Plugin Goals)

A **goal** is executed as:

```bash
mvn plugin:goal
```

### Example: Compiler Plugin

```bash
mvn compiler:compile
```

* `compiler` → plugin
* `compile` → goal

---

### Example: Clean Plugin

```bash
mvn clean:clean
```

Same as:

```bash
mvn clean
```

---

### Example: Surefire Plugin

```bash
mvn surefire:test
```

Same as:

```bash
mvn test
```

---

## 8️⃣ Common Plugin Goals Used in DevOps

| Plugin   | Goal    | Command       |
| -------- | ------- | ------------- |
| compiler | compile | `mvn compile` |
| surefire | test    | `mvn test`    |
| jar      | jar     | `mvn package` |
| deploy   | deploy  | `mvn deploy`  |

---

## 9️⃣ Lifecycle vs Direct Goal (Difference)

### Lifecycle Phase (Recommended)

```bash
mvn package
```

✔ Safe
✔ Predictable
✔ CI/CD friendly

---

### Direct Goal (Advanced)

```bash
mvn jar:jar
```

❌ Skips tests
❌ Can break pipeline logic

---

## 🔟 Useful Maven Options (DevOps Must-Know)

### Batch Mode (CI)

```bash
mvn -B clean package
```

---

### Skip Tests

```bash
mvn clean package -DskipTests
```

---

### Debug Mode

```bash
mvn clean package -X
```

---

### Offline Mode

```bash
mvn -o package
```

---

## 1️⃣1️⃣ Typical CI/CD Maven Command

```bash
mvn -B clean test package
```

Or:

```bash
mvn -B clean deploy
```

---

## 1️⃣2️⃣ Real DevOps Flow Example

```text
Git Commit
   ↓
CI Tool (Jenkins)
   ↓
mvn clean test package
   ↓
Artifact Created
   ↓
Upload to Nexus
```

---

## Interview One-Liners ⭐

> **A Maven goal is a task executed by a plugin, and a Maven phase is a step in the build lifecycle.**

> **Running a Maven phase automatically triggers all previous phases in the lifecycle.**

---

## Final Summary (Easy to Remember)

```text
mvn clean → cleanup
mvn compile → compile code
mvn test → run tests
mvn package → create artifact
mvn install → local repo
mvn deploy → remote repo
```

---

