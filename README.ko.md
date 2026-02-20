# 실시간 메시징 서비스  
### Angular + .NET 8 + SignalR + AWS (이벤트 기반 아키텍처)

[English](README.md)
---

## Executive Summary

다음과 같은 요소를 기반으로 **Production-Grade 실시간 메시징 플랫폼**을 설계 및 구현:

- ⚡ WebSocket 기반 실시간 통신 (SignalR)
- 📨 이벤트 기반 영속성 처리 (SQS → Lambda → DynamoDB)
- 👥 1:1 및 그룹 채팅 지원
- 📎 S3 Presigned URL 기반 안전한 파일 업로드
- 🔄 메시지 수정 / 삭제 / 읽음 처리 라이프사이클 지원
- ☁ AWS Elastic Beanstalk 기반 배포
- 📈 수평 확장이 가능한 백엔드 아키텍처

> 실시간 사용자 경험과 안정적인 클라우드 이벤트 처리 구조를 결합한 시스템

---

## 상위 아키텍처 (High-Level Architecture)

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

### 핵심 설계 원칙

- 실시간 전송 계층과 데이터 저장 계층을 분리
- FIFO 큐를 사용하여 메시지 순서 보장
- 비결합(Decoupled) 아키텍처를 통한 확장성 확보
- 최소 지연(Low Latency) 최적화

---

## 기술 스택 (Technical Stack)

### Frontend
- Angular 12
- SignalR JavaScript Client

### Backend
- .NET 8 (ASP.NET Core)
- SignalR Hub

### AWS 서비스
- Amazon SQS (Standard + FIFO)
- AWS Lambda
- DynamoDB
- Elastic Beanstalk (EC2 + ALB)
- Amazon S3 (Presigned URL)
- CloudWatch (로그 및 모니터링)

---

## 시스템 규모 (System Scale)

| 항목 | 규모 |
|------|------|
| 기관 | 병원 2곳 |
| 사용자 수 | 수백 명 |
| 일 평균 메시지 수 | 수백 ~ 수천 건 |
| 평균 응답 속도 | 실시간 (WebSocket 기반) |

---

# 상세 설계 (클릭하여 펼치기)

---

<details>
<summary><strong>아키텍처 다이어그램 & 메시지 흐름</strong></summary>

```
+----------------+
|   Angular App  |
| (SignalR Client)|
+--------+-------+
         |
         | 1. WebSocket 연결
         |
+--------v-------+
|  SignalR Hub    |
| (.NET API on    |
|  AWS Beanstalk) |
+--------+-------+
         |
         | 2. AWS SQS로 메시지 전송
         |
+--------v-------+
|    AWS SQS     |
+--------+-------+
         |
         | 3. Lambda 트리거
         |
+--------v-------+
|    AWS Lambda  |
+--------+-------+
         |
         | 4. DynamoDB 저장
         |
+--------v-------+
|    DynamoDB    |
+----------------+
```
<img width="600" height="400" alt="chat2" src="https://github.com/user-attachments/assets/bb9c2a21-03a5-4709-b039-cf62acf5889c" />


### 동작 흐름

1. Angular 클라이언트가 SignalR Hub에 WebSocket으로 연결
2. 서버가 메시지를 수신 후 SQS에 전송
3. Lambda가 트리거됨
4. Lambda가 DynamoDB에 메시지 저장
5. DynamoDB가 채팅 이력 저장소 역할 수행

</details>

---
<details>
<summary><strong>결과 예시</strong></summary>
<img width="300" height="300" alt="Screenshot 2026-02-19 at 23 31 35" src="https://github.com/user-attachments/assets/6d0d1721-33b1-4d8a-842d-2e2d91a290a2" />
<img width="300" height="400" alt="Screenshot 2026-02-19 at 23 33 32" src="https://github.com/user-attachments/assets/4ef9b1d7-8885-428b-8977-6cd3bb7d232e" />
<img width="300" height="500" alt="Screenshot 2026-02-19 at 23 33 32" src="https://github.com/user-attachments/assets/837a11d4-ea10-424e-b61a-68959be2fe57" />
<img width="300" height="450" alt="chat3" src="https://github.com/user-attachments/assets/1928cdb8-b62c-436a-9804-e7e932f15dcf" />
<img width="300" height="450" alt="chat4" src="https://github.com/user-attachments/assets/7c69b171-78d6-4234-a470-f7ee0e241f02" />

</details>

---
<details>
<summary><strong>핵심 기능 (Core Features)</strong></summary>

### 메시징 기능

- 실시간 메시지 송수신
- 메시지 수정(Edit)
- 메시지 삭제(Delete)
- 읽음 처리(Read Receipt)
- 그룹 채팅
- 온라인 상태 업데이트

### 채팅 라이프사이클

- 1:1 또는 그룹 채팅방 생성
- 채팅 도중 사용자 추가
- 로그인 시 채팅 목록 로드 (읽지 않은 메시지 포함)
- 양방향 채팅 자동 생성
- 전체 채팅 이력 조회

### 파일 업로드

- 파일은 Amazon S3에 저장
- DB에는 Presigned URL만 저장
- 클라이언트에서 직접 업로드 (보안 유지)

</details>

---

<details>
<summary><strong>성능 최적화 전략</strong></summary>

### 실시간 통신 최적화

- WebSocket + SignalR 사용
- HTTP Polling 오버헤드 제거
- 낮은 지연 시간 유지

### 저장소 최적화

- DynamoDB (Key-Value 모델)
  - 메시지 구조에 적합
  - 복잡한 Join 불필요
  - 수평 확장 가능

### 오프라인 메시지 처리

- 메시지는 DynamoDB에 영속 저장
- 접속하지 않은 사용자의 메시지는 `PendingMessage`에 임시 저장
- 사용자가 재접속 시:
  - 즉시 메시지 전달
  - 사용자 경험 연속성 유지

### 저장 처리 분리

- Hub에서 DB 직접 저장하지 않음
- SQS → Lambda로 비동기 처리
- Hub는 가볍고 빠르게 유지

</details>

---

<details>
<summary><strong>AWS 리소스 설계</strong></summary>

### SQS 큐

- `chat-room-queue`
- `chat-message-queue.fifo`

FIFO 설정:
- `MessageGroupId = CaseId`
- 채팅방 단위 메시지 순서 보장

### Lambda 함수

- `chat-room-to-db`
- `chat-message-to-db`

### DynamoDB

- Chat Room 테이블
- Chat Message 테이블
- 인덱스 구성:
  - 사용자 기반 조회
  - 채팅방 기반 조회
  - 이력 조회 최적화

</details>

---

<details>
<summary><strong>기술적 의사결정 (Technical Decisions)</strong></summary>

### 왜 SignalR인가?

- .NET 환경에서 WebSocket 기본 지원
- 연결 관리 단순화
- 실시간 Push 모델 제공

### 왜 SQS FIFO인가?

- 메시지 순서 보장
- 메시지 정합성 유지
- 중복 및 순서 오류 방지

### 왜 DynamoDB를 사용했는가?

- 메시지 서비스는 Key-Value 접근 패턴에 적합
- 복잡한 RDB Join 불필요
- 높은 쓰기 확장성
- 낮은 지연 시간

### 왜 S3를 사용하는가?

- 파일은 외부 스토리지에 저장
- DB에는 Presigned URL만 저장
- 저장 비용 절감
- 확장성 확보

</details>

---

<details>
<summary><strong>배포 아키텍처</strong></summary>

- Elastic Beanstalk 환경 구성
- EC2 Auto Scaling
- Application Load Balancer
- Blue/Green 배포 가능
- CloudWatch 기반 로그 및 모니터링

</details>

---

<details>
<summary><strong>프로젝트 임팩트 (Impact)</strong></summary>

### 제품 관점

- 완전한 그룹 채팅 기능 구현
- Chat History 전용 탭 구현
  - 사용자 이름, 소속, 날짜 기준 필터링
  - 메시지 내용 키워드 검색
- 사용성 및 메시지 추적성 향상

### 엔지니어링 관점

- 실시간 + 이벤트 기반 하이브리드 구조
- 병원 환경에 적합한 확장 가능한 메시징 시스템
- FIFO 기반 메시지 정합성 보장
- DynamoDB 활용으로 DB 복잡도 감소
- 관심사 분리(Clean Architecture) 유지

</details>

---

# 🏁 결론

이 프로젝트는:

- 실시간 WebSocket 아키텍처 설계
- 이벤트 기반 클라우드 시스템 설계
- FIFO 기반 메시지 정합성 보장
- 메시징 워크로드에 최적화된 DynamoDB 모델링
- 확장 가능한 AWS 기반 운영 환경

즉각적인 사용자 경험과 안정적인 백엔드 처리를 동시에 만족하는  
Production 수준의 메시징 플랫폼.
