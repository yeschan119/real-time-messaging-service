# Real-Time Messaging Service  
### Angular + .NET 8 + SignalR + AWS (Event-Driven Architecture)

---

## 🚀 Executive Summary

Designed and implemented a **production-grade real-time messaging platform** with:

- ⚡ WebSocket-based real-time communication (SignalR)
- 📨 Event-driven persistence (SQS → Lambda → DynamoDB)
- 👥 1:1 and Group Chat support
- 📎 Secure file uploads via S3 Presigned URL
- 🔄 Edit / Delete / Read Receipt lifecycle
- ☁ Fully deployed on AWS Elastic Beanstalk
- 📈 Horizontally scalable backend architecture

> Real-time user experience + asynchronous, resilient cloud persistence.

---

## 🏗 High-Level Architecture

```
Angular (SignalR Client)
        ↓ WebSocket
.NET SignalR Hub (Elastic Beanstalk)
        ↓
     Amazon SQS
        ↓
     AWS Lambda
        ↓
     DynamoDB
```

### Design Principles

- Separate **real-time delivery** from **data persistence**
- Use FIFO queues to guarantee message order
- Ensure system resiliency via decoupled architecture
- Enable horizontal scalability

---

## 🛠 Technical Stack

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

# 🔍 Detailed Design (Click to Expand)

---

<details>
<summary><strong>📈 Architecture Diagram & Flow</strong></summary>

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

### Flow Description

1. Angular client connects to SignalR Hub via WebSocket.
2. Server receives message and pushes to AWS SQS.
3. Lambda is triggered.
4. Lambda persists data to DynamoDB.
5. DynamoDB serves as chat history source.

</details>

---

<details>
<summary><strong>✨ Core Features</strong></summary>

### Real-Time Capabilities

- Instant message delivery
- Message edit & delete
- Read receipts
- Group chat
- Online presence updates

### Chat Lifecycle

- Create chat room (1:1 or group)
- Add participants dynamically
- Load chat list on login (including unread count)
- Bi-directional conversation creation
- Retrieve full chat history
- Real-time synchronization of all state changes

### File Upload

- S3 Presigned URL generation
- Secure direct client upload
- File metadata stored in message record

</details>

---

<details>
<summary><strong>📘 AWS Resource Design</strong></summary>

### Queues

- `chat-room-queue`
- `chat-message-queue.fifo`

FIFO queue configuration:
- `MessageGroupId = CaseId`
- Guarantees ordered processing per conversation

### Lambda Functions

- `chat-room-to-db`
- `chat-message-to-db`

### DynamoDB Tables

- Chat Room archive
- Chat Message archive
- Indexed for query by user and conversation

</details>

---

<details>
<summary><strong>🧠 Backend Hub Design (SignalR)</strong></summary>

### Connection Management

- In-memory `ConcurrentDictionary`
- Tracks active user connections
- Enables targeted message broadcasting

### Message Processing Strategy

When a message is sent:

1. Send immediately to connected recipients.
2. Push event to FIFO SQS queue.
3. Lambda persists message asynchronously.
4. Sender notified of delivery/read status updates.

### Key Architectural Decisions

- Separate transport layer (WebSocket) from storage layer.
- Use FIFO queues to avoid out-of-order message storage.
- Avoid direct DB writes inside Hub to minimize latency.
- Keep Hub lightweight and responsive.

### Concurrency & Scalability

- Thread-safe connection storage.
- Designed to scale horizontally under Elastic Beanstalk.
- Load balancer handles multi-instance distribution.

</details>

---

<details>
<summary><strong>🌐 Deployment Architecture</strong></summary>

- Elastic Beanstalk environment (`rts-server-prod`)
- EC2 Auto Scaling
- Application Load Balancer
- Blue/Green deployment supported
- CloudWatch logging & monitoring

</details>

---

<details>
<summary><strong>🔮 Future Improvements</strong></summary>

- Dead Letter Queue (DLQ) integration
- Message deduplication strategy
- Distributed cache for presence tracking
- WebSocket scaling via Redis backplane
- Message encryption at rest & in transit enhancement

</details>

---

# 🎯 Engineering Highlights

- WebSocket + Event-Driven hybrid architecture
- FIFO-based message ordering guarantee
- Decoupled persistence pipeline
- Real-time UX + resilient backend design
- Cloud-native scalability
- Production-ready AWS deployment

---

# 🏁 Conclusion

This project demonstrates:

- Real-time communication system design
- Event-driven cloud architecture
- Scalable messaging infrastructure
- Asynchronous persistence strategy
- Production-level backend engineering

Built for performance, scalability, and reliability in a cloud-native environment.
