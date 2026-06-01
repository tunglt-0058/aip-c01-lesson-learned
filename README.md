# AIP-C01 — Lesson Learned

> Study notes for the **AWS Certified Generative AI Developer – Professional (AIP-C01)** exam.
> **Unofficial**, not affiliated with AWS. All practice questions and case studies are **original works**. See [DISCLAIMER](./DISCLAIMER.md).

**🌐 Language:** **English** · [Tiếng Việt](./README.vi.md) · [日本語](./README.ja.md)

---

## What is this exam?

AIP-C01 is AWS's newest **Professional**-level AI certification (released Nov 2025). It validates the ability to take Generative AI to **production**: integrating Foundation Models (FMs), RAG, vector stores, security, cost optimization, and operations.

| Item | Value |
|---|---|
| Exam code | AIP-C01 |
| Level | Professional |
| Passing score | 750 / 1000 |
| Style | Pass/Fail, heavily scenario-based |

### Content domain weightings

| Domain | Weight |
|---|---|
| **D1** — Foundation Model Integration, Data Management & Compliance | **31%** |
| **D2** — Implementation & Integration | **26%** (D1 + D2 = 57%) |
| **D3** — AI Safety, Security & Governance | 20% |
| **D4** — Operational Efficiency & Optimization | 12% |
| **D5** — Testing, Validation & Troubleshooting | 11% |

---

## Repository structure

```
.
├── README.md            # English (default)  + language switcher
├── README.vi.md         # Vietnamese landing
├── README.ja.md         # Japanese landing
├── DISCLAIMER.md
├── LICENSE
├── en/                  # 🇬🇧 English
│   ├── 01-basic-knowledge/   (index + 7 service categories)
│   ├── 02-case-studies/
│   └── 03-practice-exam/
├── vi/                  # 🇻🇳 Vietnamese
│   ├── 01-basic-knowledge/
│   ├── 02-case-studies/
│   └── 03-practice-exam/
├── ja/                  # 🇯🇵 Japanese
└── assets/aws-icons/    # official AWS Architecture Icons for diagrams
```

## Where to start

**1. 📚 Basic Knowledge** — concepts by service category ([en](./en/01-basic-knowledge/) · [vi](./vi/01-basic-knowledge/) · [ja](./ja/01-basic-knowledge/))

### 🗺️ Overview mindmap — all 7 service categories

A bird's-eye view of every service covered in Basic Knowledge (01 → 07):

```mermaid
mindmap
  root((AIP-C01<br/>Services))
    01 Amazon Bedrock
      Access models
        Bedrock core - FM API
        Sync vs Streaming
        On-Demand vs Provisioned
        Fine-tuning
        Model Evaluation
        Cross-Region Inference
      Knowledge and RAG
        Knowledge Bases - RAG
        Chunking strategies
        Metadata filtering
        Data Automation
      Control and safety
        Guardrails - content
        Prompt Management
        Prompt Flows
      Agents
        Bedrock Agents
        AgentCore
          Runtime - Memory - Gateway
          Identity - Browser - Code Interpreter
          Observability - Policy
    02 SageMaker
      Data prep
        Ground Truth
        Data Wrangler
        Processing
        Feature Store
      Train and build
        SageMaker AI
        JumpStart
        Training
        Pipelines - CICD
      Govern and explain
        Model Registry
        Model Monitor
        Clarify - SHAP PDP bias
      Deploy and workspace
        Neo - edge
        Unified Studio - lakehouse
    03 AI ML Supporting
      Text and search
        Comprehend and Medical
        Kendra
      Voice Vision Chat
        Lex
        Rekognition
        Transcribe
        Polly
      Document
        Textract
      Quality control
        A2I - human in loop
      Amazon FMs
        Titan
        Nova
        Nova Multimodal Embeddings
    04 Amazon Q
      Q Developer
        Code and security scan
        IaC and CLI
        Troubleshoot CloudWatch
      Q Business
        Managed RAG
        Data Connectors - ACL
        Plugins
        Conversation memory
      Q Apps - templates
    05 Data and Analytics
      Streaming
        Kinesis Streams Firehose Analytics
        MSK - Kafka
      ETL
        Glue - Data Catalog
        EMR - Spark
      Vector and search
        OpenSearch - k-NN
        S3 Vectors
      Query and visualize
        Athena - SQL on S3
        QuickSight - BI
    06 Integration and Orchestration
      Orchestration and events
        Step Functions
        EventBridge
      Compute and API
        Lambda and Durable Functions
        API Gateway - REST WebSocket
        AppSync - GraphQL
      Config
        AppConfig - feature flags
      Agent framework
        Strands SDK
        Multi-agent patterns
        MCP
    07 Security and Governance
      Access
        IAM - least privilege
      Data protection
        KMS - CMK
        Secrets Manager
        Parameter Store
        VPC Endpoint
      Monitoring
        CloudTrail - who did what
        CloudWatch - metrics
      Model safety
        Bedrock Guardrails
        Macie - PII in S3
      Cost and governance
        Cost Explorer
        Well-Architected GenAI Lens
```

**2. 🧩 Case Studies** ([en](./en/02-case-studies/) · [vi](./vi/02-case-studies/) · [ja](./ja/02-case-studies/))

### 🧩 Case Studies — the 14 architectures at a glance

Each case follows the same template: use case + requirements → Mermaid diagram → design rationale → trade-offs → lesson learned.

| # | Core concept | File |
|---|---|---|
| 1 | End-to-end GenAI architecture (FM → RAG → CRM → global network → GenAIOps) | [en](./en/02-case-studies/case-01-multinational-financial-chatbot.md) · [vi](./vi/02-case-studies/case-01-multinational-financial-chatbot.md) · [ja](./ja/02-case-studies/case-01-multinational-financial-chatbot.md) |
| 2 | Multi-FM abstraction layer + resilience + GenAIOps (healthcare) | [en](./en/02-case-studies/case-02-healthcare-document-analysis.md) · [vi](./vi/02-case-studies/case-02-healthcare-document-analysis.md) · [ja](./ja/02-case-studies/case-02-healthcare-document-analysis.md) |
| 3 | Multi-modal pipeline (text/image/audio/tabular) + data fusion | [en](./en/02-case-studies/case-03-insurance-claims-multimodal.md) · [vi](./vi/02-case-studies/case-03-insurance-claims-multimodal.md) · [ja](./ja/02-case-studies/case-03-insurance-claims-multimodal.md) |
| 4 | Multi-tiered vector DB — the right store per data type | [en](./en/02-case-studies/case-04-legal-search-assistant.md) · [vi](./vi/02-case-studies/case-04-legal-search-assistant.md) · [ja](./ja/02-case-studies/case-04-legal-search-assistant.md) |
| 5 | Model-control framework (Prompt Mgmt + Guardrails + JSON Schema) | [en](./en/02-case-studies/case-05-financial-customer-service-platform.md) · [vi](./vi/02-case-studies/case-05-financial-customer-service-platform.md) · [ja](./ja/02-case-studies/case-05-financial-customer-service-platform.md) |
| 6 | Tiered model deployment (Lambda / Bedrock PT / SageMaker) | [en](./en/02-case-studies/case-06-ecommerce-tiered-deployment.md) · [vi](./vi/02-case-studies/case-06-ecommerce-tiered-deployment.md) · [ja](./ja/02-case-studies/case-06-ecommerce-tiered-deployment.md) |
| 7 | Legacy connectivity + data sovereignty + GenAI gateway | [en](./en/02-case-studies/case-07-enterprise-genai-integration.md) · [vi](./vi/02-case-studies/case-07-enterprise-genai-integration.md) · [ja](./ja/02-case-studies/case-07-enterprise-genai-integration.md) |
| 8 | Sync/async/streaming + multi-layer fallback | [en](./en/02-case-studies/case-08-healthcare-flexible-interaction.md) · [vi](./vi/02-case-studies/case-08-healthcare-flexible-interaction.md) · [ja](./ja/02-case-studies/case-08-healthcare-flexible-interaction.md) |
| 9 | Multi-agent orchestration (Strands / Agent Squad) + Amazon Q | [en](./en/02-case-studies/case-09-insurance-claims-agentic.md) · [vi](./vi/02-case-studies/case-09-insurance-claims-agentic.md) · [ja](./ja/02-case-studies/case-09-insurance-claims-agentic.md) |
| 10 | Defense-in-depth + adversarial testing | [en](./en/02-case-studies/case-10-financial-safety-controls.md) · [vi](./vi/02-case-studies/case-10-financial-safety-controls.md) · [ja](./ja/02-case-studies/case-10-financial-safety-controls.md) |
| 11 | Network isolation + access control + PII + anonymization | [en](./en/02-case-studies/case-11-healthcare-data-security.md) · [vi](./vi/02-case-studies/case-11-healthcare-data-security.md) · [ja](./ja/02-case-studies/case-11-healthcare-data-security.md) |
| 12 | Responsible AI (LLM-as-a-judge + bias monitoring + Audit Manager) | [en](./en/02-case-studies/case-12-responsible-ai-fairness.md) · [vi](./vi/02-case-studies/case-12-responsible-ai-fairness.md) · [ja](./ja/02-case-studies/case-12-responsible-ai-fairness.md) |
| 13 | Tiered usage + intelligent routing + batch/caching | [en](./en/02-case-studies/case-13-cost-effective-model-selection.md) · [vi](./vi/02-case-studies/case-13-cost-effective-model-selection.md) · [ja](./ja/02-case-studies/case-13-cost-effective-model-selection.md) |
| 14 | Capacity planning + GenAI-specific auto-scaling + Inferentia | [en](./en/02-case-studies/case-14-resource-allocation-fm-workloads.md) · [vi](./vi/02-case-studies/case-14-resource-allocation-fm-workloads.md) · [ja](./ja/02-case-studies/case-14-resource-allocation-fm-workloads.md) |

**3. ✅ Practice Exam** ([en](./en/03-practice-exam/) · [vi](./vi/03-practice-exam/) · [ja](./ja/03-practice-exam/))

### ✅ The 20 questions at a glance

What each original question tests and the AWS services it touches (*(2)* = Select TWO):

| # | What it tests | Key AWS services & concepts | Link |
|---|---|---|---|
| 1 | RAG result reranking *(2)* | Knowledge Bases hybrid search, Bedrock reranker, OpenSearch | [Q1](./en/03-practice-exam/questions.md#question-1--rag-the-best-result-sinks-to-the-bottom-select-two) |
| 2 | Real-time & resilient KB sync | S3 Event Notifications, SQS, Lambda, Ingest/Delete API | [Q2](./en/03-practice-exam/questions.md#question-2--real-time-and-resilient-knowledge-base-sync) |
| 3 | Analyze images/video, least overhead | Bedrock multimodal FMs, Step Functions, QuickSight | [Q3](./en/03-practice-exam/questions.md#question-3--analyze-imagesvideo-with-the-least-effort) |
| 4 | Order a model-evaluation workflow | metrics → dataset → A/B test → quality gates (Step Functions) → report | [Q4](./en/03-practice-exam/questions.md#question-4--order-the-model-replacement-evaluation-workflow) |
| 5 | Enforce guardrails on every call | IAM condition key `bedrock:GuardrailIdentifier` | [Q5](./en/03-practice-exam/questions.md#question-5--enforce-guardrails-on-every-call) |
| 6 | Stop generation at a phrase | stop sequences (inference parameter) | [Q6](./en/03-practice-exam/questions.md#question-6--stop-generation-at-a-specific-phrase) |
| 7 | LLM endpoint optimization *(2)* | max sequence length, tensor parallelism, DJL, SageMaker | [Q7](./en/03-practice-exam/questions.md#question-7--optimize-llm-endpoint-resource-utilization-select-two) |
| 8 | Real-time streaming to a web UI | API Gateway WebSocket, Lambda, Bedrock streaming API, Prompt Management | [Q8](./en/03-practice-exam/questions.md#question-8--stream-real-time-suggestions-to-a-web-editor) |
| 9 | Prompt governance + long-term logging *(2)* | Bedrock Prompt Management, model invocation logging, S3 Object Lock | [Q9](./en/03-practice-exam/questions.md#question-9--prompt-governance--long-term-compliance-logging-select-two) |
| 10 | Deploy a Python agent to AgentCore *(2)* | AgentCore SDK `@app.entrypoint`, starter toolkit | [Q10](./en/03-practice-exam/questions.md#question-10--deploy-a-python-agent-to-agentcore-runtime-select-two) |
| 11 | Source lineage for generated content *(2)* | metadata tagging, AWS Glue Data Catalog | [Q11](./en/03-practice-exam/questions.md#question-11--source-lineage-for-generated-content-select-two) |
| 12 | RAG silent failure after a deploy | embedding model version / vector-space mismatch | [Q12](./en/03-practice-exam/questions.md#question-12--rag-silent-failure-after-a-code-update) |
| 13 | Monitor KB ingestion errors | Knowledge Base logging, CloudWatch Logs Insights | [Q13](./en/03-practice-exam/questions.md#question-13--monitor-document-ingestion-into-a-knowledge-base) |
| 14 | Amazon Q Developer productivity *(2)* | code generation/refactor, test generation in CI/CD | [Q14](./en/03-practice-exam/questions.md#question-14--maximize-productivity-with-amazon-q-developer-select-two) |
| 15 | SageMaker inference type for image gen | Asynchronous vs Real-time / Serverless / Batch Transform | [Q15](./en/03-practice-exam/questions.md#question-15--choose-the-right-sagemaker-inference-type-for-image-generation) |
| 16 | Large-scale infrequent vector search | Amazon S3 Vectors vs OpenSearch / RDS / DynamoDB | [Q16](./en/03-practice-exam/questions.md#question-16--large-scale-vector-search-infrequent-cheapest) |
| 17 | Which guardrail rule fired | guardrail tracing, GuardrailPolicyType vs GuardrailContentSource | [Q17](./en/03-practice-exam/questions.md#question-17--identify-which-guardrail-rule-blocked-the-request) |
| 18 | Secure auth + IdP, no long-lived creds *(2)* | Amazon Cognito (OIDC), IAM Identity Center (SAML) | [Q18](./en/03-practice-exam/questions.md#question-18--secure-auth-idp-federation-no-long-lived-credentials-select-two) |
| 19 | Peak throttling, same FM, cheapest | Cross-Region Inference vs Provisioned Throughput | [Q19](./en/03-practice-exam/questions.md#question-19--peak-hour-throttling-same-fm-cheapest) |
| 20 | Redact PII before search | Amazon Comprehend (PII redaction) + Amazon Kendra | [Q20](./en/03-practice-exam/questions.md#question-20--redact-pii-before-feeding-it-to-search) |

## Content status

| Section | vi | en | ja |
|---|---|---|---|
| Basic Knowledge (7 service categories) | ✅ | ✅ | ✅ |
| Case Studies | ✅ 14 | ✅ | ✅ |
| Practice Exam | ✅ 20 | ✅ | ✅ |

> 🔲 not started · 🚧 in progress · ✅ drafted

## License

- **Content** (notes, case studies, questions): [CC BY 4.0](./LICENSE)
- **Code** (Mermaid, config snippets): MIT

See [DISCLAIMER.md](./DISCLAIMER.md) for copyright notes.
