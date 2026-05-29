# 📚 Basic Knowledge — theo nhóm Service

[← Về trang chủ](../../README.vi.md)

Phần này tổ chức kiến thức **theo nhóm dịch vụ AWS** (thay vì theo 5 domain hàn lâm) để dễ tra cứu: muốn ôn Bedrock thì vào đúng 1 chỗ, muốn ôn bảo mật thì vào đúng 1 chỗ.

> ⚠️ Đề thi vẫn tính điểm theo **5 domain**. Để không mất tính bám đề, mỗi service đều gắn tag *domain liên quan*, và có [bảng cross-map nhóm service ↔ domain](#bản-đồ-nhóm-service--5-exam-domain) bên dưới.

## Mindmap 7 nhóm service

```mermaid
mindmap
  root((AIP-C01<br/>Services))
    01 Amazon Bedrock
      Bedrock core - FM API
      Knowledge Bases - RAG
      Guardrails
      Prompt Management & Flows
      Agents & AgentCore
      Model Evaluation
      Data Automation
    02 SageMaker
      SageMaker AI
      Model Registry
      Pipelines
      Data Wrangler & Processing
      Model Monitor & Clarify
    03 AI/ML Supporting
      Comprehend
      Textract
      Transcribe & Translate
      Rekognition
      Kendra
    04 Amazon Q
      Q Business
      Q Developer
      Q in QuickSight
    05 Data & Analytics
      S3
      OpenSearch vector
      Aurora pgvector
      DynamoDB
      Glue & Data Quality
      Kinesis & MSK
    06 Integration & Orchestration
      Lambda
      Step Functions
      API Gateway & AppSync
      EventBridge SQS SNS
      AppConfig
    07 Security & Governance
      IAM & KMS
      VPC Endpoint PrivateLink
      CloudTrail & CloudWatch
      Macie
      Secrets Manager
```

## 7 nhóm service

| # | Nhóm | Gồm gì (ví dụ tiêu biểu) | File |
|---|---|---|---|
| 01 | **Amazon Bedrock Services** | Bedrock core, Knowledge Bases, Guardrails, Prompt Management/Flows, Agents/AgentCore, Model Evaluation, Data Automation | [→](./01-amazon-bedrock-services.md) |
| 02 | **SageMaker Services** | SageMaker AI, Model Registry, Pipelines, Data Wrangler, Processing, Model Monitor, Clarify, JumpStart | [→](./02-sagemaker-services.md) |
| 03 | **AI/ML Supporting Services** | Comprehend, Textract, Transcribe, Translate, Rekognition, Polly, Kendra | [→](./03-ai-ml-supporting-services.md) |
| 04 | **Amazon Q Services** | Q Business, Q Developer, Q in QuickSight | [→](./04-amazon-q-services.md) |
| 05 | **Data & Analytics Services** | S3, OpenSearch (vector), Aurora pgvector, DynamoDB, Glue (Data Quality), Kinesis, MSK, Athena | [→](./05-data-analytics-services.md) |
| 06 | **Integration & Orchestration Services** | Lambda, Step Functions, API Gateway, AppSync, EventBridge, SQS, SNS, AppConfig | [→](./06-integration-orchestration-services.md) |
| 07 | **Security & Governance Services** | IAM, KMS, VPC Endpoint/PrivateLink, CloudTrail, CloudWatch, Macie, Secrets Manager | [→](./07-security-governance-services.md) |

## Bản đồ: nhóm service ↔ 5 exam domain

Bảng này cho thấy mỗi nhóm service "phục vụ" domain nào nhiều nhất — để khi ôn theo service vẫn biết mình đang phủ phần điểm nào.

| Nhóm service \ Domain | D1 (31%) | D2 (26%) | D3 (20%) | D4 (12%) | D5 (11%) |
|---|:---:|:---:|:---:|:---:|:---:|
| 01 Bedrock | ●●● | ●●● | ●● | ●● | ●● |
| 02 SageMaker | ●● | ●● | ● | ● | ●● |
| 03 AI/ML Supporting | ●● | ●● | ● | | |
| 04 Amazon Q | ● | ●● | ● | | |
| 05 Data & Analytics | ●●● | ● | ● | ● | |
| 06 Integration & Orchestration | ●● | ●●● | | ●● | ● |
| 07 Security & Governance | ● | | ●●● | ● | ● |

> ● phụ · ●● vừa · ●●● chính. (Mức độ là ước lượng định hướng ôn tập, không phải con số chính thức của AWS.)
> Nhắc lại 5 domain: **D1** FM Integration & Data · **D2** Implementation & Integration · **D3** AI Safety/Security/Governance · **D4** Operational Efficiency · **D5** Testing/Validation/Troubleshooting.

## Cách đọc mỗi "service card"

Mỗi service trình bày ngắn gọn, đời thường (không hàn lâm) theo khuôn:

- **Một câu là gì** — giải thích bằng ví von dễ hình dung.
- **Giải quyết bài toán gì** — dùng để làm gì trong thực tế.
- **Khi nào dùng** — dấu hiệu trong đề/dự án để chọn nó.
- **Khi nào KHÔNG dùng / dễ nhầm với** — ranh giới với service khác.
- **Liên quan domain thi** — tag D1…D5.
- **⚠️ Điểm phải nhớ** — bẫy hay gặp.
- **🧪 Ví dụ 1 dòng** — minh hoạ nhanh.
