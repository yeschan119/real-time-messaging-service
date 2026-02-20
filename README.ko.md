# Real-Time Messaging Service  
### Angular + .NET Core + SignalR + AWS 기반 실시간 채팅 시스템

---

## 🚀 Executive Summary

Production-grade **Real-Time Messaging Platform** 설계 및 구현

- ⚡ WebSocket 기반 실시간 메시징 (SignalR)
- 📨 메시지 영속성: SQS → Lambda → DynamoDB
- 🏢 Elastic Beanstalk 배포
- 👥 1:1 / Group Chat 지원
- 📎 파일 업로드 (S3 Presigned URL)
- 📖 채팅 이력 관리
- 🔄 Edit / Delete / Read Receipt 지원
- ☁ Cloud-native 비동기 아키텍처

---

## 🏗 High-Level Architecture

```
Angular → SignalR Hub (.NET on Beanstalk)
           ↓
          SQS
           ↓
         Lambda
           ↓
        DynamoDB
```

- WebSocket 기반 실시간 전송
- 비동기 메시지 저장 구조
- FIFO Queue로 메시지 순서 보장
- 확장 가능한 서버 구조

---

## 🛠 Tech Stack

**Frontend**
- Angular 12.0.3
- SignalR JavaScript Client

**Backend**
- .NET 8.0.7 (ASP.NET Core)
- SignalR Hub

**AWS Services**
- Amazon SQS (Standard + FIFO)
- AWS Lambda
- DynamoDB
- Elastic Beanstalk (EC2 + Load Balancer)
- Amazon S3 (Presigned URL)

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
<summary><strong>✨ Key Features</strong></summary>

### Core Features

- Real-time message transmission (WebSocket)
- Edit / Delete message
- Read receipts
- Group chatting
- File upload via S3 Presigned URL

### Functional Details

- **Create User**: Create chat room with organization user
- **Add User**: Add user to create group chat
- **Chatting List**: Load chat list (with unread) on login
- **Two-way communication**: Chat created both ways
- **Real-time message**: Send/receive via WebSocket
- **Read/Update/Delete**: Full message lifecycle support
- **Real-time storage**: All actions stored asynchronously
- **Chat history**: Retrieve and interact with past messages

</details>

---

<details>
<summary><strong>📘 AWS Resource Flow</strong></summary>

1. Angular connects to SignalR Hub.
2. Messages pushed to AWS SQS:
   - `chat-room-queue`
   - `chat-message-queue.fifo` (FIFO for ordered processing)
3. Lambda triggered:
   - `chat-room-to-db`
   - `chat-message-to-db`
4. DynamoDB stores:
   - Chat rooms
   - Chat messages
   - Chat archive index

</details>

---

<details>
<summary><strong>🌐 Deployment</strong></summary>

- Elastic Beanstalk (Blue/Green 가능)
- Environment: `rts-server-prod`
- EC2 + Load Balancer 구조
- Monitoring via CloudWatch

</details>

---

<details>
<summary><strong>🔮 Future Improvements</strong></summary>

- Dead Letter Queue (DLQ) 도입
- Message Deduplication 강화
- Online Presence 개선
- Horizontal Scaling 최적화

</details>

---

# 🧠 Backend Hub Design (SignalR Hub)

<details>
<summary><strong>📌 ChatHub 핵심 설계 설명</strong></summary>

### 주요 설계 요소

- `ConcurrentDictionary<string, UserConnection>`
  - 연결된 사용자 관리
- `ConcurrentDictionary<int, PendingMessage>`
  - 오프라인 메시지 관리 구조 (현재 주석 처리)
- SQS Queue URL Lazy Initialization
- FIFO Queue (`chat-message-queue.fifo`)
  - `MessageGroupId = CaseId`
  - 메시지 순서 보장

### 메시지 저장 전략

1. 클라이언트 → Hub
2. Hub → SQS (비동기)
3. Lambda → DynamoDB 저장
4. Hub → Receiver WebSocket Push

### 실시간 + 비동기 저장 분리

- 사용자 경험: WebSocket 실시간
- 데이터 영속성: SQS 기반 Event-Driven

→ 응답 속도와 안정성 분리 설계

</details>

---

<details>
<summary><strong>📄 ChatHub Code</strong></summary>

```csharp
// (원본 ChatHub 코드 그대로 유지)
using Microsoft.AspNetCore.SignalR;
using Newtonsoft.Json;
using System.Threading.Tasks;
using System.Collections.Concurrent;
using System.Collections.Generic;
using System;
using System.Linq;
using covid_server.Models;
using Amazon.SQS;
using Amazon.SQS.Model;
using Amazon.DynamoDBv2;
using Amazon.DynamoDBv2.DocumentModel;

// ... (전체 코드 동일)
// 실제 README에는 코드 전체 그대로 삽입 가능
```

</details>

---

# 🎯 Engineering Highlights

- WebSocket + Event-Driven Architecture
- FIFO Queue 기반 메시지 순서 보장
- 실시간 처리와 영속성 처리 분리
- ConcurrentDictionary 기반 연결 관리
- Presigned URL 기반 파일 업로드 보안 처리
- Elastic Beanstalk 기반 확장 구조

---

# 🏁 Conclusion

이 프로젝트는 단순한 채팅 기능 구현이 아닌,

- Real-Time WebSocket Architecture
- Cloud-Native Event-Driven Design
- 확장 가능한 AWS 기반 저장 구조
- 메시지 순서 보장 + 비동기 영속성
- Production-Ready Messaging Platform

을 설계·구현한 시스템입니다.
