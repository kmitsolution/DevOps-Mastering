# Maven Installation Guide (Windows, Linux, macOS)

## Prerequisite: Java Installation (Mandatory)

👉 **Java must be installed before Maven**

Maven is written in Java, so **Java is mandatory**.

---

## 1️⃣ Maven Installation on Linux – Ubuntu

### Step 1: Install Java (OpenJDK 21)

```bash
sudo apt update
sudo apt install openjdk-21-jdk -y
```

Verify Java:

```bash
java -version
```

---

### Step 2: Install Maven (Using apt – Recommended)

```bash
sudo apt install maven -y
```

Verify Maven:

```bash
mvn --version
```

---

### Alternative (Manual Installation from Apache Website)

1. Go to **Apache Maven official website**
   👉 [https://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi)
```
   Example:
    wget https://dlcdn.apache.org/maven/maven-3/3.9.12/binaries/apache-maven-3.9.12-bin.tar.gz
```
3. Download the **latest Binary tar.gz**

4. Extract:

```bash
tar -xvzf apache-maven-*.tar.gz
```

4. Move to `/opt`:

```bash
mkdir /opt/maven
sudo mv apache-maven-* /opt/maven
```

5. Set environment variables:

```bash
export MAVEN_HOME=/opt/maven/apache-maven-3.9.12
export PATH=$MAVEN_HOME/bin:$PATH
```

6. Reload:

```bash
source ~/.bashrc
```

7. Verify:

```bash
mvn --version
```

---

## 2️⃣ Maven Installation on Linux – Red Hat / CentOS / Rocky / Alma

### Step 1: Install Java

```bash
sudo yum install java-21-openjdk-devel -y
```

Verify:

```bash
java -version
```

---

### Step 2: Install Maven (YUM)

```bash
sudo yum install maven -y
```

Verify:

```bash
mvn --version
```

---

### Alternative: Manual Installation

(Same Apache tar file steps as Ubuntu)

---

## 3️⃣ Maven Installation on macOS

### Step 1: Install Java

Using Homebrew:

```bash
brew install openjdk@21
```

Verify:

```bash
java -version
```

---

### Step 2: Install Maven (Homebrew)

```bash
brew install maven
```

Verify:

```bash
mvn --version
```

---

### Alternative (Manual Installation)

* Download from: [https://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi)
* Extract and set `MAVEN_HOME` and `PATH`

---

## 4️⃣ Maven Installation on Windows

### Step 1: Install Java (Required)

Download **OpenJDK 21** from:

* [https://adoptium.net/](https://adoptium.net/)

Install and set:

* `JAVA_HOME`
* Add `%JAVA_HOME%\bin` to **PATH**

Verify:

```cmd
java -version
```

---

### Step 2: Download Maven (Windows)

Download Maven from **official site**:
👉 [https://maven.apache.org/download.cgi](https://maven.apache.org/download.cgi)

* Download **Binary zip archive**
* Extract to:

```text
C:\Program Files\Apache\maven
```

---

### Step 3: Set Environment Variables (Windows)

Add system variables:

```text
MAVEN_HOME = C:\Program Files\Apache\maven
```

Add to PATH:

```text
%MAVEN_HOME%\bin
```

Restart Command Prompt.

---

### Step 4: Verify Maven

```cmd
mvn --version
```

---

## 5️⃣ Maven Verification Output (Expected)

```text
Apache Maven 3.x.x
Maven home: /usr/share/maven
Java version: 21
OS name: Linux / Windows / Mac OS X
```

---

## DevOps Best Practice ⭐

* Always install **Java first**
* Use **package manager** in CI servers
* Verify with:

```bash
java -version
mvn --version
```

* Cache `.m2` directory in CI pipelines

---

## Interview One-Liner ⭐

> **To install Maven, first install Java, then install Maven using OS package managers or download it from the Apache Maven website and verify using `mvn --version`.**
