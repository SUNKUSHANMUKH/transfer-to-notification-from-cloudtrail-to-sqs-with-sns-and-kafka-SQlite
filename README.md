
# **CloudTrail → SQS → Fluentd (EC2) → Kafka Pipeline**

This repository contains configuration files and scripts required to stream **AWS CloudTrail logs** into **Apache Kafka** using **Fluentd (td-agent)** running on an EC2 instance.

This pipeline ensures:

* **Scalable ingestion** of CloudTrail logs
* **Reliable delivery** through S3 + SQS
* **Real-time streaming** into Kafka for analytics, SIEM, monitoring, or custom microservices

---

## **📌 Architecture Overview**

```
CloudTrail → S3 Bucket → SQS Queue → Fluentd (EC2) → Kafka Cluster
```

### Components:

* **AWS CloudTrail** – Generates logs and stores them in S3
* **S3 Bucket** – Stores CloudTrail logs in compressed .gz format
* **SQS Queue** – Receives object-created notifications from CloudTrail
* **Fluentd on EC2** – Fetches logs from S3, parses them, and sends them to Kafka
* **Kafka Cluster** – Final destination for streaming CloudTrail events

---

## **📁 Files Included in This Package**

| File                     | Description                                                                       |
| ------------------------ | --------------------------------------------------------------------------------- |
| `td-agent.conf`          | Fluentd config for ingesting SQS messages, fetching S3 logs, and pushing to Kafka |
| `install_td_agent.sh`    | Shell script to install Fluentd and required plugins                              |
| `start_td_agent.sh`      | Starts and enables the Fluentd service                                            |
| `sqs_policy.json`        | Example SQS access policy allowing CloudTrail to send notifications               |
| `kafka_consumer_test.sh` | Sample Kafka consumer command for validation                                      |
| `README.md`              | This documentation                                                                |

---

## **🚀 1. Prerequisites**

Before using this project, ensure you have:

### **AWS Resources**

* S3 bucket for CloudTrail logs
* SQS Standard Queue
* CloudTrail Trail configured to send notifications to SQS
* EC2 instance (Amazon Linux 2 recommended) with:

  * IAM Role permissions:

    * `s3:GetObject`
    * `sqs:ReceiveMessage`
    * `sqs:DeleteMessage`
    * `sqs:ChangeMessageVisibility`

### **Kafka Cluster**

You may use:

* Amazon MSK
* Confluent Cloud
* Self-hosted Kafka cluster

You’ll need:

* Bootstrap servers
* Kafka topic (default: `aws-cloudtrail-logs`)
* Security settings (PLAINTEXT / SSL / SASL / IAM)

---

## **⚙️ 2. Install Fluentd on EC2**

SSH into the EC2 instance and run:

```bash
sudo bash install_td_agent.sh
```

This installs:

* td-agent (Fluentd)
* fluent-plugin-s3
* fluent-plugin-sqs
* fluent-plugin-kafka

---

## **📝 3. Configure Fluentd**

Replace `/etc/td-agent/td-agent.conf` with the provided `td-agent.conf`.

Before running, edit the config and update values:

* `<YOUR_SQS_QUEUE_URL>`
* `<YOUR_AWS_REGION>`
* `<YOUR_CLOUDTRAIL_BUCKET_NAME>`
* `<YOUR_KAFKA_BOOTSTRAP_BROKERS>`
* `<YOUR_SASL_USERNAME>`
* `<YOUR_SASL_PASSWORD>`

Validate config syntax:

```bash
sudo td-agent --dry-run -c /etc/td-agent/td-agent.conf
```

---

## **▶️ 4. Start Fluentd Service**

```bash
sudo bash start_td_agent.sh
```

Check logs:

```bash
tail -f /var/log/td-agent/td-agent.log
```

---

## **🔍 5. Testing & Validation**

### **Step 1 — Trigger CloudTrail Events**

Perform any AWS action such as:

* Starting/stopping EC2
* Creating an IAM role
* Uploading an S3 file

### **Step 2 — Check SQS**

Messages should appear briefly and then be consumed by Fluentd.

### **Step 3 — Check Kafka for Logs**

```bash
bash kafka_consumer_test.sh
```

---

## **🛠️ Troubleshooting**

### 🔸 No logs in Kafka?

* Check `td-agent.log`
* Ensure EC2 has access to SQS, S3, and Kafka SG rules
* Verify Kafka credentials and protocol settings

### 🔸 Fluentd crashes?

Run:

```bash
sudo td-agent --dry-run -c td-agent.conf
```

### 🔸 SSL/SASL errors?

* Verify CA cert path
* Validate SASL username/password
* Check MSK IAM authentication settings (if used)

---

## **📦 Customization & Extensions**

I can also generate:

* Terraform automation for full deployment
* CloudFormation templates
* Systemd supervision or log rotation configs
* A Lambda-based alternative to Fluentd

---
