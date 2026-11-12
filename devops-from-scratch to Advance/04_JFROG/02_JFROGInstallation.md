**installation guide** for setting up **JFrog Artifactory using Docker Compose on Ubuntu 22.04**.

---

# **Install JFrog Artifactory using Docker Compose on Ubuntu 22.04**

## **📌 Pre-requisites**
✅ **Ubuntu EC2 instance** (Recommended: `t2.medium` with **4GB RAM**)  
✅ **Security Group:** Open ports **8081, 8082**  
✅ **Docker & Docker Compose installed**  

---

## **Step 1: Change Hostname**
Set a custom hostname for your instance:
```bash
sudo hostnamectl set-hostname Artifactory
```
---

## **Step 2: Update System**
```bash
sudo apt update
sudo apt install docker.io -y
```
---

## **Step 3: Install Docker Compose**
```bash
sudo apt install docker-compose -y
```
Verify installation:
```bash
docker-compose --version
```
---

## **Step 4: Create `docker-compose.yml` File**
This file contains the configuration for installing Artifactory.

```bash
sudo vi docker-compose.yml
```
**Note**:- you can change the artifactory version from this link <a href=https://jfrog.com/download-legacy/> Jfrog </a>

**Paste the following YAML content:**
```yaml
version: "3.3"
services:
  artifactory-service:
    image: docker.bintray.io/jfrog/artifactory-oss:7.49.6
    container_name: artifactory
    restart: always
    networks:
      - ci_net
    ports:
      - 8081:8081
      - 8082:8082
    volumes:
      - artifactory:/var/opt/jfrog/artifactory

volumes:
  artifactory:
networks:
  ci_net:
```
**Save and exit:** Press `Esc`, then type `:wq!` and press `Enter`.

---

## **Step 5: Deploy Artifactory with Docker Compose**
Run the following command to start the Artifactory container:
```bash
sudo docker-compose up -d
```
✅ The `-d` flag runs the container in **detached mode**.

---

## **Step 6: Verify Artifactory is Running**
Check the logs to ensure Artifactory started successfully:
```bash
sudo docker-compose logs --follow
```
If everything is fine, you will see logs confirming Artifactory is running.

---

## **Step 7: Access Artifactory in the Browser**
🔹 Open your browser and go to:  
```bash
http://<your-public-dns>:8081
```
**Example:**  
```bash
http://ec2-xx-xx-xx-xx.compute-1.amazonaws.com:8081
```

---

## **Step 8: Default Login Credentials**
| Username  | Password  |
|-----------|-----------|
| `admin`   | `password` |

---

## **🎯 Summary**
1. **Updated system & installed Docker Compose**  
2. **Created `docker-compose.yml`**  
3. **Deployed Artifactory using Docker Compose**  
4. **Accessed Artifactory on port `8081`**  

---

## **📌 Troubleshooting**
### **1️⃣ Check Running Containers**
```bash
docker ps
```
### **2️⃣ Restart Artifactory**
```bash
sudo docker-compose restart
```
### **3️⃣ Check Logs**
```bash
sudo docker-compose logs --tail 50
```
### **4️⃣ Fix Port Conflicts**
```bash
sudo netstat -tulnp | grep 8081
```
If another service is using port 8081, stop it or modify the compose file.

---
