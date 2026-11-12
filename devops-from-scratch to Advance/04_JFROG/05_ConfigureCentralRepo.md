# JFrog Project Documentation

## 1. Create a New Maven Project
To create a new Maven project named **testproj**, use the following command:

```sh
mvn archetype:generate -DgroupId=com.raja -DartifactId=testproj -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

## 2. Add Dependency in `pom.xml`
Modify the `pom.xml` file to include the dependency that is available in JFrog Artifactory but not in the local repository:

```xml
<dependencies>
    <dependency>
        <groupId>com.raman</groupId>
        <artifactId>myproj</artifactId>
        <version>1.0</version>
    </dependency>
</dependencies>
```

## 3. Configure JFrog Repository in Maven Settings
Since the dependency is not in the local repository, configure Maven to use JFrog's virtual repository.

1. Copy the **Virtual Repository URL** from **SetMe** in JFrog.
2. Open the Maven settings file located at `/usr/share/maven/conf/settings.xml`.
3. Add the following `<mirrors>` section:

```xml
<mirrors>
   <mirror>
      <id>central</id>
      <mirrorOf>*</mirrorOf>
      <name>JFrog Virtual Repository</name>
      <url>http://44.204.196.116:8081/artifactory/virutal-repo</url>
    </mirror>
</mirrors>
```

## 4. Deploy the Code to Local Repository
Before deploying, add the **distribution management** settings in `pom.xml`:

```xml
<distributionManagement>
    <repository>
        <id>jfrog-local</id>
        <url>http://44.204.196.116:8081/artifactory/libs-release-local</url>
    </repository>
</distributionManagement>
```

Then, run the following Maven commands:

```sh
mvn clean package
mvn deploy
```

## 5. Verify Deployment
Check the JFrog **local repository** (e.g., `libs-release-local`) to confirm that `testproj` has been successfully deployed.

---
