# 🧩 Case Studies

[← Về trang chủ](../../README.vi.md)

Các case study **viết lại hoàn toàn (original)** từ pattern kiến trúc đã học, nhằm luyện tư duy "đọc tình huống → chọn giải pháp AWS phù hợp".

> ⚠️ **Bản quyền:** không copy nội dung/ảnh từ AWS Skill Builder. Mỗi case study là tình huống tự viết (khác ngành, khác bối cảnh, khác số liệu); mọi diagram do tác giả tự vẽ.

## Danh sách (14 case studies)

| # | Case study | Concept chính | Domain | File |
|---|---|---|---|---|
| 01 | Chatbot CSKH ngân hàng đa quốc gia | Kiến trúc GenAI end-to-end (FM → RAG → CRM → mạng toàn cầu → GenAIOps) | D1–D4 | [→](./case-01-multinational-financial-chatbot.md) |
| 02 | Phân tích hồ sơ y tế (HIPAA) | Abstraction layer đa-FM + resilience + GenAIOps | D1,D2,D4,D5 | [→](./case-02-healthcare-document-analysis.md) |
| 03 | Bồi thường bảo hiểm đa phương thức | Pipeline multi-modal (text/ảnh/audio/bảng) + data fusion | D1,D2,D5 | [→](./case-03-insurance-claims-multimodal.md) |
| 04 | Trợ lý tra cứu pháp lý | Vector DB đa tầng — chọn kho cho từng loại dữ liệu | D1,D2,D3 | [→](./case-04-legal-search-assistant.md) |
| 05 | Nền tảng CSKH tài chính | Khung điều khiển model (Prompt Mgmt + Guardrails + JSON Schema) | D2,D3,D4 | [→](./case-05-financial-customer-service-platform.md) |
| 06 | CSKH TMĐT co giãn | Tiered model deployment (Lambda/Bedrock PT/SageMaker) | D2,D4,D5 | [→](./case-06-ecommerce-tiered-deployment.md) |
| 07 | Tích hợp GenAI doanh nghiệp tài chính | Kết nối legacy + chủ quyền dữ liệu + GenAI gateway | D2,D3,D4 | [→](./case-07-enterprise-genai-integration.md) |
| 08 | Trợ lý AI y tế linh hoạt | Sync/async/streaming + fallback nhiều lớp | D2,D4,D5 | [→](./case-08-healthcare-flexible-interaction.md) |
| 09 | Bồi thường bảo hiểm agentic | Multi-agent orchestration (Strands/Agent Squad) + Amazon Q | D2,D4,D5 | [→](./case-09-insurance-claims-agentic.md) |
| 10 | Kiểm soát an toàn AI tài chính | Defense-in-depth + adversarial testing | D3,D5 | [→](./case-10-financial-safety-controls.md) |
| 11 | Bảo mật & riêng tư dữ liệu y tế | Network isolation + access control + PII + anonymization | D1,D3 | [→](./case-11-healthcare-data-security.md) |
| 12 | Đánh giá fairness chatbot y tế | Responsible AI (LLM-as-a-judge + bias monitoring + Audit Manager) | D3,D5 | [→](./case-12-responsible-ai-fairness.md) |
| 13 | Chọn model tối ưu chi phí | Tiered usage + intelligent routing + batch/caching | D1,D4 | [→](./case-13-cost-effective-model-selection.md) |
| 14 | Phân bổ tài nguyên workload FM | Capacity planning + auto scaling đặc thù GenAI + Inferentia | D2,D4 | [→](./case-14-resource-allocation-fm-workloads.md) |

> Dùng [`_case-template.md`](./_case-template.md) làm khuôn cho case study mới.

## Quy ước hình ảnh

- Diagram nhúng bằng **Mermaid** (render thẳng trên GitHub) — ưu tiên, không cần file ảnh.
- Nếu cần ảnh PNG/SVG: đặt trong [`./assets/`](./assets/), tự vẽ bằng draw.io + AWS Architecture Icons.
- Đặt tên file ảnh theo case: `case-01-architecture.png`.
