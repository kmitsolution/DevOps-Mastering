
## 🔔 Amazon EventBridge & Event Bus (AWS)

Amazon EventBridge is a **serverless event routing service** that receives events from AWS services, custom apps, or SaaS apps and routes them to targets like **SNS, Lambda, SQS, Step Functions**, etc.

Think of it as:

> **IF event happens → THEN do something automatically**

---

## 🧱 Event Bus Types

### **1️⃣ Default Event Bus**

* Automatically created in every AWS account
* Receives **AWS service events**
* Example: EC2 state changes, S3 events, EBS events

✅ Most AWS exam questions use **default event bus**

---

### **2️⃣ Custom Event Bus**

* Created by you
* Used for **custom application events**
* You send events using `PutEvents` API

Use case:

* Microservices sending business events
* Cross-account event routing

---

## 🔍 Event Types

### **AWS Service Events**

* Generated automatically
* Example:

  * EC2 instance state change
  * Auto Scaling events

### **Custom Events**

* Sent by your application
* Example:

```json
{
  "source": "my.app",
  "detail-type": "orderCreated",
  "detail": {
    "orderId": "123"
  }
}
```

---

## 🖼️ Architecture: EC2 → EventBridge → SNS → Lambda

![Image](https://miro.medium.com/1%2A-TTLTKq2b8FA217WJboOlw.png)

![Image](https://d2908q01vomqb2.cloudfront.net/887309d048beef83ad3eabf2a79a64a389ab1c9f/2021/11/16/DBBLOG-1756-image001.png)

---

## 🎯 Real-Time Use Case (EXAM + REAL LIFE)

**Requirement:**

> Notify and process events when an EC2 instance **starts, stops, or terminates**

Flow:

```
EC2 State Change
   ↓
EventBridge (Default Event Bus)
   ↓
Rule
   ↓
SNS Topic
   ↓
Lambda Function
```

---

## 🛠️ STEP-BY-STEP (AWS Console)

---

### ✅ **Step 1: Create SNS Topic**

1. Go to **SNS → Topics**
2. Click **Create topic**
3. Type: `Standard`
4. Name: `ec2-state-change-topic`
5. Create

📌 Note the **Topic ARN**

---

### ✅ **Step 2: Create Lambda Function**

1. Go to **Lambda → Create function**
2. Name: `ProcessEC2StateChange`
3. Runtime: Python 3.10
4. Role:

   * Allow **SNS invoke Lambda**
   * Allow **CloudWatch Logs**

Sample Lambda code:

```python
def lambda_handler(event, context):
    print("Received event:", event)
    return {"status": "processed"}
```

---

### ✅ **Step 3: Subscribe Lambda to SNS**

1. Go to **SNS → Subscriptions**
2. Create subscription:

   * Protocol: `Lambda`
   * Endpoint: `ProcessEC2StateChange`
3. Save

---

### ✅ **Step 4: Create EventBridge Rule (IMPORTANT)**

1. Go to **EventBridge → Rules**
2. Click **Create rule**

#### Rule Details

* Name: `ec2-state-change-rule`
* Event bus: **default**
* Rule type: **Rule with an event pattern**

---

### ✅ **Step 5: Event Pattern for EC2 State Change**

Choose:

* **Event source:** AWS services
* **AWS service:** EC2
* **Event type:** EC2 Instance State-change Notification

OR use **custom JSON** 👇 (EXAM FAVORITE)

```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["running", "stopped", "terminated"]
  }
}
```

---

### ✅ **Step 6: Add Target (SNS)**

* Target type: **SNS topic**
* Topic: `ec2-state-change-topic`

---

### ✅ **Step 7: Permissions (Auto-handled)**

EventBridge automatically adds permission to publish to SNS.

---

## 🧪 Test the Flow

1. Start or stop an EC2 instance
2. EventBridge receives event
3. SNS publishes message
4. Lambda executes

🎉 End-to-end automation achieved!

---

## 📘 What the EXAM LOVES

✔ Default Event Bus = AWS service events
✔ Event pattern filtering
✔ Decoupled architecture
✔ Serverless, no polling
✔ SNS fan-out + Lambda processing

---

## ⚠️ Common Exam Traps

❌ Using CloudWatch Events (old name)
❌ Using SQS polling instead of EventBridge
❌ Hardcoding EC2 instance IDs when pattern matching is enough

---

## 🧠 One-Line Exam Answer

> **Use Amazon EventBridge default event bus with an EC2 state-change rule, target SNS, and invoke Lambda for processing.**

---

