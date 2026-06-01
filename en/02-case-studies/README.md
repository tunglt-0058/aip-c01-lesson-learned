# 🧩 Case Studies

[← Home](../../README.md)

14 architecture case studies, each following the same template: use case summary + requirements table → Mermaid architecture diagram → design rationale (why each service) → alternatives & trade-offs → lesson learned.

## List (14 case studies)

| # | Case study | Core concept | Domain | File |
|---|---|---|---|---|
| 01 | Multinational bank CS chatbot | End-to-end GenAI architecture (FM → RAG → CRM → global network → GenAIOps) | D1–D4 | [→](./case-01-multinational-financial-chatbot.md) |
| 02 | Healthcare document analysis (HIPAA) | Multi-FM abstraction layer + resilience + GenAIOps | D1,D2,D4,D5 | [→](./case-02-healthcare-document-analysis.md) |
| 03 | Multi-modal insurance claims | Multi-modal pipeline (text/image/audio/tabular) + data fusion | D1,D2,D5 | [→](./case-03-insurance-claims-multimodal.md) |
| 04 | Legal research assistant | Multi-tiered vector DB — the right store per data type | D1,D2,D3 | [→](./case-04-legal-search-assistant.md) |
| 05 | Financial customer-service platform | Model-control framework (Prompt Mgmt + Guardrails + JSON Schema) | D2,D3,D4 | [→](./case-05-financial-customer-service-platform.md) |
| 06 | Scalable e-commerce support | Tiered model deployment (Lambda/Bedrock PT/SageMaker) | D2,D4,D5 | [→](./case-06-ecommerce-tiered-deployment.md) |
| 07 | Enterprise GenAI integration (finance) | Legacy connectivity + data sovereignty + GenAI gateway | D2,D3,D4 | [→](./case-07-enterprise-genai-integration.md) |
| 08 | Flexible healthcare AI assistant | Sync/async/streaming + multi-layer fallback | D2,D4,D5 | [→](./case-08-healthcare-flexible-interaction.md) |
| 09 | Agentic insurance claims | Multi-agent orchestration (Strands/Agent Squad) + Amazon Q | D2,D4,D5 | [→](./case-09-insurance-claims-agentic.md) |
| 10 | Financial AI safety controls | Defense-in-depth + adversarial testing | D3,D5 | [→](./case-10-financial-safety-controls.md) |
| 11 | Healthcare data security & privacy | Network isolation + access control + PII + anonymization | D1,D3 | [→](./case-11-healthcare-data-security.md) |
| 12 | Healthcare chatbot fairness | Responsible AI (LLM-as-a-judge + bias monitoring + Audit Manager) | D3,D5 | [→](./case-12-responsible-ai-fairness.md) |
| 13 | Cost-effective model selection | Tiered usage + intelligent routing + batch/caching | D1,D4 | [→](./case-13-cost-effective-model-selection.md) |
| 14 | FM-workload resource allocation | Capacity planning + GenAI-specific auto-scaling + Inferentia | D2,D4 | [→](./case-14-resource-allocation-fm-workloads.md) |

> Use [`_case-template.md`](./_case-template.md) as the template for a new case study.
