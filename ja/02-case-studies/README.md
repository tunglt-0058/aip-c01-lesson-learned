# 🧩 ケーススタディ

[← ホーム](../../README.md)

14 件のアーキテクチャケーススタディ。各々同じテンプレートに従う: ユースケース要約 + 要件表 → Mermaid アーキテクチャ図 → design rationale（各サービスの理由）→ 代替案とトレードオフ → 学び。

## 一覧（14 ケーススタディ）

| # | ケーススタディ | 中心概念 | ドメイン | ファイル |
|---|---|---|---|---|
| 01 | 多国籍銀行の CS chatbot | エンドツーエンド GenAI アーキテクチャ（FM → RAG → CRM → グローバル網 → GenAIOps） | D1–D4 | [→](./case-01-multinational-financial-chatbot.md) |
| 02 | 医療文書分析（HIPAA） | マルチ FM abstraction layer + resilience + GenAIOps | D1,D2,D4,D5 | [→](./case-02-healthcare-document-analysis.md) |
| 03 | マルチモーダル保険請求 | マルチモーダルパイプライン（text/画像/audio/表）+ data fusion | D1,D2,D5 | [→](./case-03-insurance-claims-multimodal.md) |
| 04 | 法務リサーチアシスタント | マルチ階層 vector DB — データ型ごとに正しいストア | D1,D2,D3 | [→](./case-04-legal-search-assistant.md) |
| 05 | 金融カスタマーサービス基盤 | モデル制御フレームワーク（Prompt Mgmt + Guardrails + JSON Schema） | D2,D3,D4 | [→](./case-05-financial-customer-service-platform.md) |
| 06 | スケーラブル EC サポート | 階層化モデルデプロイ（Lambda/Bedrock PT/SageMaker） | D2,D4,D5 | [→](./case-06-ecommerce-tiered-deployment.md) |
| 07 | 全社的 GenAI 統合（金融） | レガシー接続 + データ主権 + GenAI gateway | D2,D3,D4 | [→](./case-07-enterprise-genai-integration.md) |
| 08 | 柔軟なヘルスケア AI アシスタント | Sync/async/streaming + 多層 fallback | D2,D4,D5 | [→](./case-08-healthcare-flexible-interaction.md) |
| 09 | Agentic 保険請求 | Multi-agent orchestration（Strands/Agent Squad）+ Amazon Q | D2,D4,D5 | [→](./case-09-insurance-claims-agentic.md) |
| 10 | 金融 AI 安全制御 | Defense-in-depth + adversarial testing | D3,D5 | [→](./case-10-financial-safety-controls.md) |
| 11 | ヘルスケアデータセキュリティ & プライバシー | Network isolation + access control + PII + anonymization | D1,D3 | [→](./case-11-healthcare-data-security.md) |
| 12 | ヘルスケア chatbot の公平性 | Responsible AI（LLM-as-a-judge + bias 監視 + Audit Manager） | D3,D5 | [→](./case-12-responsible-ai-fairness.md) |
| 13 | コスト効率的なモデル選定 | Tiered usage + intelligent routing + batch/caching | D1,D4 | [→](./case-13-cost-effective-model-selection.md) |
| 14 | FM ワークロードのリソース配分 | Capacity planning + GenAI 固有 auto-scaling + Inferentia | D2,D4 | [→](./case-14-resource-allocation-fm-workloads.md) |

> 新しいケーススタディには [`_case-template.md`](./_case-template.md) をテンプレートとして使う。
