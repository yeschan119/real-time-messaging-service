# Real-Time Messaging Service  
### Angular + .NET 8 + SignalR + AWS (Event-Driven Architecture)

[한국어 🇰🇷](README.ko.md)
---

## Executive Summary

Designed and implemented a **production-grade real-time messaging platform** with:

- ⚡ WebSocket-based real-time communication (SignalR)
- 📨 Event-driven persistence (SQS → Lambda → DynamoDB)
- 👥 1:1 and Group Chat support
- 📎 Secure file uploads via S3 Presigned URL
- 🔄 Edit / Delete / Read Receipt lifecycle
- ☁ Fully deployed on AWS Elastic Beanstalk
- 📈 Horizontally scalable backend architecture

> Real-time UX combined with resilient cloud-native event processing.

---

## High-Level Architecture

```
Angular (SignalR Client)
        ↓ WebSocket
.NET SignalR Hub (Elastic Beanstalk)
        ↓
     Amazon SQS (FIFO)
        ↓
     AWS Lambda
        ↓
     DynamoDB
```

### Core Design Principles

- Separate real-time transport from data persistence
- Guarantee message order using FIFO queues
- Ensure scalability through decoupled architecture
- Optimize for low latency

---

## Technical Stack

### Frontend
- Angular 12
- SignalR JavaScript Client

### Backend
- .NET 8 (ASP.NET Core)
- SignalR Hub

### AWS Services
- Amazon SQS (Standard + FIFO)
- AWS Lambda
- DynamoDB
- Elastic Beanstalk (EC2 + ALB)
- Amazon S3 (Presigned URLs)
- CloudWatch (Monitoring & Logs)

---

## System Scale

| Metric | Value |
|--------|-------|
| Organizations | 2 Hospitals |
| Users | Hundreds |
| Daily Messages | Hundreds ~ Thousands |
| Average Response | Real-time (WebSocket) |

---

# Detailed Sections (Click to Expand)

---

<details>
<summary><strong>Architecture Diagram & Flow</strong></summary>

```
+----------------+
|   Angular App  |
| (SignalR Client)|
+--------+-------+
         |
         | 1. Connect via WebSocket
         |
+--------v-------+
|  SignalR Hub    |
| (.NET API on    |
|  AWS Beanstalk) |
+--------+-------+
         |
         | 2. Save Message to AWS SQS
         |
+--------v-------+
|    AWS SQS     |
+--------+-------+
         |
         | 3. Trigger Lambda
         |
+--------v-------+
|    AWS Lambda  |
+--------+-------+
         |
         | 4. Save to DynamoDB
         |
+--------v-------+
|    DynamoDB    |
+----------------+
```

<img width="600" height="400" alt="chat2" src="https://github.com/user-attachments/assets/bb9c2a21-03a5-4709-b039-cf62acf5889c" />

### Flow Description

1. Angular client connects to SignalR Hub via WebSocket.
2. Server receives message and pushes to AWS SQS.
3. Lambda is triggered.
4. Lambda persists data to DynamoDB.
5. DynamoDB serves as chat history source.

</details>

---

<details>
<summary><strong>Results - example</strong></summary>
<img width="300" height="300" alt="Screenshot 2026-02-19 at 23 31 35" src="https://github.com/user-attachments/assets/6d0d1721-33b1-4d8a-842d-2e2d91a290a2" />
<img width="300" height="400" alt="Screenshot 2026-02-19 at 23 33 32" src="https://github.com/user-attachments/assets/4ef9b1d7-8885-428b-8977-6cd3bb7d232e" />
<img width="300" height="500" alt="Screenshot 2026-02-19 at 23 33 32" src="https://github.com/user-attachments/assets/837a11d4-ea10-424e-b61a-68959be2fe57" />
<img width="300" height="450" alt="chat3" src="https://github.com/user-attachments/assets/1928cdb8-b62c-436a-9804-e7e932f15dcf" />
<img width="300" height="450" alt="chat4" src="https://github.com/user-attachments/assets/7c69b171-78d6-4234-a470-f7ee0e241f02" />

</details>

---
<details>
<summary><strong>Core Features</strong></summary>

### Messaging

- Real-time send/receive
- Edit message
- Delete message
- Read receipts
- Group chat
- Online presence updates

### Chat Lifecycle

- Create chat room (1:1 or group)
- Add users dynamically
- Load chat list on login (including unread)
- Bi-directional chat initialization
- Retrieve chat history

### File Upload

- Store files in Amazon S3
- Persist only Presigned URL in message record
- Secure direct upload from client

</details>

---

<details>
<summary><strong>Performance Optimization</strong></summary>

### Real-Time Communication

- WebSocket + SignalR for minimal latency
- Avoid HTTP polling overhead

### Storage Optimization

- DynamoDB (Key-Value model)
  - Better fit for message workload
  - Avoid relational join overhead
  - Scales horizontally

### Offline Message Handling

- Messages persisted in DynamoDB
- Temporarily stored in `PendingMessage` in memory
- When user reconnects:
  - Pending messages are delivered immediately
  - Ensures seamless user experience

### Decoupled Persistence

- Hub does NOT write directly to DB
- SQS → Lambda handles storage asynchronously
- Keeps Hub lightweight and responsive

</details>

---

<details>
<summary><strong>AWS Resource Design</strong></summary>

### Queues

- `chat-room-queue`
- `chat-message-queue.fifo`

FIFO Configuration:
- `MessageGroupId = CaseId`
- Ensures strict ordering within each chat session

### Lambda Functions

- `chat-room-to-db`
- `chat-message-to-db`

### DynamoDB

- Chat Room table
- Chat Message table
- Indexed for:
  - User-based query
  - Conversation-based query
  - Archive retrieval

</details>

---

<details>
<summary><strong>Technical Decisions</strong></summary>

### Why SignalR?

- Native WebSocket support in .NET
- Simplified connection management
- Real-time push model

### Why SQS FIFO?

- Guarantee message ordering
- Ensure message consistency
- Prevent duplicate/out-of-order persistence

### Why DynamoDB instead of RDBMS?

- Message workload fits Key-Value access pattern
- No complex joins required
- Higher write scalability
- Lower latency for chat queries

### Why S3 for File Storage?

- Store files externally
- Persist only Presigned URL in DB
- Reduce DB storage cost
- Improve scalability

</details>

---

<details>
<summary><strong>Deployment Architecture</strong></summary>

- Elastic Beanstalk environment
- EC2 Auto Scaling
- Application Load Balancer
- Blue/Green deployment supported
- CloudWatch logging & monitoring

</details>

---

<details>
<summary><strong>Impact</strong></summary>

### Product-Level Impact

- Implemented full Group Chat functionality
- Built dedicated Chat History tab
  - Filter conversations by:
    - User name
    - Organization
    - Date
  - Search message content by keyword
- Improved usability and message traceability

### Engineering Impact

- Real-time + event-driven hybrid architecture
- Scalable messaging system for hospital environment
- Reliable message ordering
- Reduced DB complexity by using DynamoDB
- Clean separation of concerns

</details>

---

# 🏁 Conclusion

This project demonstrates:

- Real-time WebSocket architecture
- Event-driven cloud design
- FIFO-based message consistency
- DynamoDB key-value modeling for messaging workloads
- Scalable, production-ready AWS deployment

Built to deliver instant user experience with resilient backend processing.
