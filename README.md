# AWS SNS, DLQ, and Notifications – Key Concepts

## 1. Does SNS guarantee exactly-once delivery to subscribers?

**No, SNS does not guarantee exactly-once delivery.**  
SNS guarantees **at-least-once delivery**, which means:

- A message published to an SNS topic will be delivered **at least once** to each subscribed endpoint.
- In certain scenarios (e.g., network retries, subscriber timeouts), a message may be delivered **more than once**, leading to duplicate deliveries.
- It is the **responsibility of the subscriber** to handle idempotency to ensure correct processing if the same message is received multiple times.

---

## 2. What is the purpose of the Dead-letter Queue (DLQ)?

A **Dead-letter Queue (DLQ)** is a special queue (typically an SQS queue) that stores **messages that could not be successfully processed** by their intended target after a configured number of delivery attempts.

- For **SQS**: If a consumer repeatedly fails to process a message within the `maxReceiveCount`, the message is moved to the DLQ.  
- For **SNS**: If a subscribed endpoint (e.g., Lambda, SQS) fails repeatedly to receive a message, SNS can move the message to a DLQ.  
- For **EventBridge**: Events that can’t be delivered to the target can also be sent to a DLQ for later analysis.

**Purpose:**
- Prevent message loss.  
- Help with debugging and troubleshooting failed message processing.  
- Allow reprocessing or manual handling later.

---

## 3. How to enable a notification to your email when messages are added to the DLQ?

To get an **email notification** when messages land in a DLQ, you can use **Amazon CloudWatch Alarms** with **Amazon SNS Email Subscription**. Example steps:

1. **Create an SNS Topic for Alerts**  
   - Go to **Amazon SNS** → **Topics** → Create topic.  
   - Choose “Standard”.  
   - Name it (e.g., `DLQ-Alerts`).  
   - Create the topic.

2. **Subscribe Your Email**  
   - In the SNS topic, click **Create subscription**.  
   - Protocol: `Email`  
   - Endpoint: your email address.  
   - Confirm the subscription by clicking the link in the email you receive.

3. **Create a CloudWatch Alarm on the DLQ**  
   - Go to **Amazon CloudWatch** → **Alarms** → Create alarm.  
   - Choose metric: **SQS → Queue Metrics → ApproximateNumberOfMessagesVisible** for your DLQ.  
   - Set the threshold (e.g., ≥ 1 message).  
   - For the **alarm action**, choose the SNS topic created earlier (`DLQ-Alerts`).  
   - Create the alarm.

Now, whenever messages appear in the DLQ, CloudWatch will detect it and send a notification to your email through SNS.

---

## ✅ Summary Table

| Feature                       | SNS                              | DLQ (SQS / SNS / EventBridge)                    |
|-------------------------------|-----------------------------------|-------------------------------------------------|
| Delivery Guarantee            | At-least-once                     | N/A (used for failures)                         |
| Exactly-once Delivery         | ❌ No                             | Not applicable                                 |
| DLQ Purpose                   | N/A                              | Store failed messages for later analysis       |
| Email Notification Mechanism  | Via SNS + CloudWatch Alarm       | Detects DLQ messages and sends alert          |

