### **🚀 Setting Up Local, Remote, and Virtual Repositories in JFrog Artifactory (GUI)**  

JFrog Artifactory allows you to create three types of repositories:  

1. **Local Repository** → Stores artifacts you deploy.  
2. **Remote Repository** → Caches external artifacts from repositories like Maven Central.  
3. **Virtual Repository** → Combines local and remote repositories for simplified access.  

---

## **🔹 Step 1: Log in to JFrog Artifactory**  
1. Open a browser and go to your JFrog Artifactory URL:  
   ```
   http://<your-artifactory-ip>:8081/artifactory/
   ```
2. Log in with your **admin** credentials.

---

## **🔹 Step 2: Create a Local Repository**  
1. Navigate to **Admin** → **Repositories** → **Local** → **New Local Repository**.  
2. Choose the package type **(e.g., Maven, npm, Docker, etc.).**  
3. Set a **Repository Key** (e.g., `my-local-repo`).  
4. Click **Save & Finish**.  

✅ This repository is now ready to store internally published artifacts.

---

## **🔹 Step 3: Create a Remote Repository**  
1. Navigate to **Admin** → **Repositories** → **Remote** → **New Remote Repository**.  
2. Choose the package type **(e.g., Maven, npm, PyPI, Docker, etc.).**  
3. Set a **Repository Key** (e.g., `maven-remote`).  
4. Enter the **Remote URL** (e.g., for Maven Central: `https://repo.maven.apache.org/maven2/`).  
5. Enable **Caching** (optional, but recommended for performance).  
6. Click **Save & Finish**.  

✅ This repository will proxy external repositories and cache artifacts.

---

## **🔹 Step 4: Create a Virtual Repository**  
1. Navigate to **Admin** → **Repositories** → **Virtual** → **New Virtual Repository**.  
2. Choose the package type **(e.g., Maven, npm, Docker, etc.).**  
3. Set a **Repository Key** (e.g., `maven-virtual`).  
4. Add **local and remote repositories** to the virtual repository.  
5. Click **Save & Finish**.  

✅ This repository will serve as a single entry point, combining multiple repositories.

---

## **🔹 Step 5: Test the Setup**  
1. Go to **Artifacts** in the left panel.  
2. Click your **local, remote, or virtual repository** to browse stored artifacts.  
3. Test deployment using Maven:  
   ```sh
   mvn deploy
   ```
4. Test downloading artifacts using:  
   ```sh
   mvn dependency:resolve
   ```

---

### 🎯 **Done! Your JFrog Artifactory Repositories Are Ready!**  
- **Local Repo** → Stores your own artifacts.  
- **Remote Repo** → Caches external dependencies.  
- **Virtual Repo** → Provides a unified view of multiple repositories.  

Let me know if you need more details! 🚀
