### 🚀 **Uploading a Maven Project to JFrog Artifactory**  

This guide explains how to create a Maven project, configure it for JFrog Artifactory, and deploy artifacts.

---

## **🔹 Step 1: Create a Maven Project**  
Run the following command to generate a Maven project:  

```sh
mvn archetype:generate -DgroupId=com.raman -DartifactId=myproj -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```
This will create a project with:  
- **Package name** → `com.raman`
- **Artifact ID** → `myproj`

---

## **🔹 Step 2: Build the JAR File**  
Move into the project directory and package the project:  

```sh
cd myproj
mvn package
```
This compiles the project and generates a `.jar` file in `target/`.

---

## **🔹 Step 3: Configure `pom.xml` for JFrog Artifactory**  
Modify `pom.xml` to include the Artifactory repository under `<distributionManagement>`:  

```xml
<distributionManagement>
    <repository>
        <id>repo-1</id>
        <name>localhost.localdomain-releases</name>
        <url>http://192.168.3.61:8081/artifactory/devopsschool-local-release</url>
    </repository>
    <snapshotRepository>
        <id>repo-1</id>
        <name>localhost.localdomain-snapshots</name>
        <url>http://192.168.3.61:8081/artifactory/devopsschool-local-snapshot</url>
    </snapshotRepository>
</distributionManagement>
```
- **Releases** go to `devopsschool-local-release`
- **Snapshots** go to `devopsschool-local-snapshot`

---

## **🔹 Step 4: Configure `settings.xml` for Authentication**  
Edit the Maven `settings.xml` file (`~/.m2/settings.xml`)  /usr/share/maven/conf and add credentials:  

```xml
<servers>
    <server>
      <id>repo-1</id>
      <username>admin</username>
      <password>Password@123</password>
    </server>
    <server>
      <id>snapshots</id>
      <username>admin</username>
      <password>Password@123</password>
    </server>
</servers>
```
- The `<id>` in `settings.xml` must match the one in `pom.xml`.

---

## **🔹 Step 5: Deploy the Maven Project**  
Deploy the project to JFrog Artifactory:  

```sh
mvn deploy
```
- This uploads the JAR to the Artifactory repository.

---

## **🔹 Step 6: Verify in JFrog Artifactory**  
Log in to JFrog Artifactory at `http://192.168.3.61:8081/` and check:  
- **Releases** → `devopsschool-local-release`
- **Snapshots** → `devopsschool-local-snapshot`

---

## **🔹 Step 7: Change the Version and Deploy Again**  
1. Update the `<version>` in `pom.xml`, for example:  
   ```xml
   <version>1.1</version>
   ```
2. Deploy again:  
   ```sh
   mvn deploy
   ```
3. Check if the new version appears in Artifactory.

---

## 🎯 **Done! Your Maven Project is Now in JFrog Artifactory!** 🎯  
