# 🚨 AWS Alerts to ServiceNow Incident Automation (API-Based)

This project demonstrates how AWS alerts can automatically generate ServiceNow incident tickets using Amazon EventBridge, AWS Lambda, Amazon SES, and the ServiceNow REST API. The integration is currently implemented in a single AWS account (Dev account), but it is designed to scale across multiple AWS accounts using EventBridge rules that trigger a Lambda function in the central Dev account.

---

## 📌 Use Case

We want to automatically create ServiceNow incidents when AWS alerts (e.g., EC2 or DRS issues) are triggered, using the ServiceNow **REST API** rather than relying on email or Distribution Lists (DLs).

> 🔹 **Implemented in a single AWS account (Dev)**  
> 🔹 **Easily extendable to multi-account setups by setting up EventBridge rules in each account that trigger the central Lambda in the Dev account**  
> 🔹 **Built to integrate with ServiceNow’s Incident Table via REST API**

---

## 🛠️ Architecture Overview

### (Single Account View)

```

+---------------------+        +------------------+        +--------------------+         +-------------------------+
| AWS Services (e.g.  | ----> | EventBridge Rule  | ----> | Lambda Function     | -----> | ServiceNow REST API      |
| EC2, DRS, Backup)   |       | (in Dev Account)  |       | (Dev Account)       |        | (Create Incident)        |
+---------------------+        +------------------+        +--------------------+         +-------------------------+

```

### (Multi-Account View)

```

+----------------------+        +------------------+        +--------------------+        +-------------------------+
| Other AWS Accounts   | ----> | EventBridge Rule  | ----> | Lambda (Dev Account)| ----> | ServiceNow REST API      |
| (via EventBridge)    |       | (Triggers Lambda  |       | (IAM Role Assumed)  |       | (Create Incident)        |
+----------------------+        +------------------+        +--------------------+        +-------------------------+

```

---

## 🔧 Components

### ✅ EventBridge Rule
- Triggers on specific AWS alerts (e.g., EC2, DRS, etc.).
- In a multi-account setup, each account will have its own EventBridge rule that targets the central Lambda in the Dev account.

### ✅ Lambda Function (Dev Account)
- Parses incoming CloudWatch/EventBridge alerts from multiple AWS accounts.
- Formats the payload and authenticates with the ServiceNow API to create incidents.
- Credentials are securely stored via environment variables or AWS Secrets Manager.
- **SES is configured in the Dev account** for any email-based notifications or incident creation-related communications (if applicable).
  
**IAM Role Permissions for Lambda:**
- The Lambda function must assume an IAM role with permissions to:
  - Read EventBridge events.
  - Access Secrets Manager to retrieve ServiceNow credentials.
  - Write logs to CloudWatch for debugging and monitoring purposes.
  - Optionally, send email notifications using SES.

### ✅ ServiceNow API
- Receives incident creation requests at `/api/now/table/incident`.
- Requires API authentication (username/password or OAuth token).
  
---

## 📂 Repository Structure

```

aws-alerts-to-servicenow-api/
├── lambda/
│   └── create\_incident.py        # Lambda code to post to ServiceNow
├── terraform/
│   ├── eventbridge.tf            # EventBridge rules
│   ├── iam\_roles.tf              # IAM roles and policies
│   └── lambda\_secrets.tf         # Store API credentials securely (e.g., Secrets Manager)
├── diagrams/
│   └── architecture.png          # (Optional) Visual diagram
├── README.md

````

---

## 🚀 Getting Started

### 1. Configure EventBridge Rules
- Set up EventBridge rules to capture CloudWatch alerts from services like EC2, DRS, or AWS Backup.
- In each account, configure the EventBridge rule to trigger the Lambda in the **Dev account**.

### 2. Set Up Lambda Function
- Lambda function parses the alert and formats the request to match ServiceNow's incident creation schema.
- Use environment variables or Secrets Manager to securely store ServiceNow API credentials.
- Ensure that SES is configured in the **Dev account** to handle any email notifications or communications.

### 3. Enable ServiceNow API Integration
- Work with your ServiceNow team to:
  - Get a user account with permissions to create incidents.
  - Share the ServiceNow instance URL and API endpoint path.
  - Define the required fields for the incident payload.

---

## 🧪 Sample Incident Payload

```json
{
  "short_description": "DRS Replication Failed - i-0abcdef1234567890",
  "description": "AWS DRS reported replication failure.\nInstance ID: i-0abcdef1234567890\nAccount: 123456789012\nRegion: us-east-1",
  "impact": "2",
  "urgency": "2",
  "category": "infrastructure",
  "priority": "2",
  "state": "1",
  "assigned_to": "admin"
}
````

---

## 🔐 Security

* Use **AWS Secrets Manager** to store ServiceNow credentials securely.
* Ensure Lambda is secured with **least-privilege IAM roles**.
* Enable logging and set up alerts for failed API calls to improve observability.
* Implement retries and **Dead Letter Queues (DLQ)** for failed incidents, if necessary.

---

## 📈 Future Enhancements

* **Multi-Alert Channel Notifications**  
  In addition to creating incidents in ServiceNow, you could send notifications to other channels like **Slack**, **Microsoft Teams**, or **SMS** for immediate attention. This can be achieved by integrating **AWS SES**, **SNS**, or **Lambda-triggered messaging services**. For example, you could configure Lambda to send a custom alert message to a Slack channel or Microsoft Teams when a high-severity AWS alert is triggered, ensuring the right teams are notified immediately, regardless of whether the ServiceNow incident is created.

---

## 🧠 Notes

* This project contains **no proprietary or client-specific data**.
* Designed for real-world **AWS + ITSM automation use cases**.
* Makes use of **public APIs** and common **AWS services** only.

---

### Key Updates:
- **None as of May 6, 2025**

