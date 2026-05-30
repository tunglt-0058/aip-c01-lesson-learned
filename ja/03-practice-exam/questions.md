# Practice Questions (Original)

[← Practice Exam に戻る](./README.md)

> 著者が自作した **オリジナル問題 20 問**。検証対象の *concept* から書き直し（業界・文脈・数値はどの実試験とも異なる）。**実際の試験問題ではなく、AWS Skill Builder からのコピーでもありません。** [DISCLAIMER](../../DISCLAIMER.md) を参照。
>
> **使い方:** シナリオを読む → まず自分で答えを選ぶ → 「分析と解答を見る」を開く。**核心分析・選択肢分解・試験のコツ** が正解/不正解より重要。

---

## 問 1 — RAG: 最良の結果が下に沈む（2 つ選択）

> RAG（Retrieval-Augmented Generation — 検索拡張生成）システムで非常によくある **「病気」** に触れる問題: 関連文書はひと山取れるのに、**一番良い文書が一番下に沈む**。これを **最小の手間** でトップに押し上げるには?

ある通信会社が Amazon Bedrock で RAG アプリを構築し、ネットワーク障害対応の文書を検索する。多くの関連箇所を取得できるが、技術者は「最も役立つ箇所がリストの下の方にある」と報告。会社は **最小の operational overhead** で結果の順位を改善したい。

正しい 2 つのステップは?（2 つ選択）

- **A.** Knowledge Bases で **hybrid search**（vector embeddings + keyword matching の併用）を有効化、backend に OpenSearch Serverless。
- **B.** OpenSearch に Learning to Rank プラグインを自前で導入、ユーザーの click データを収集してスコアモデルを学習。
- **C.** **Amazon Bedrock の managed reranker model（再ランキングモデル）** で意味的関連度に基づき結果を並べ替え。
- **D.** Aurora PostgreSQL + pgvector を作り、独自の類似度スコアアルゴリズムを書く。
- **E.** SageMaker JumpStart + Kendra Intelligent Ranking で独自スコアアルゴリズムを作る。

<details>
<summary>📖 分析と解答を見る</summary>

### 🎯 この問題が試す concept
**built-in の managed 機能** で RAG の **retrieval（検索）** 品質を上げる（自作しない）。Domain 1。

### 🔍 核心分析 — 「砂金採り」の問題
RAG はコーパスを検索してひと山（例 20 箇所）を拾うが、最良の答えを含む箇所が 19 番目にある。AI が上から読むと **「情報過多」** で見落とす — 古典的な *"Lost in the middle"*。

**特効薬 — 「Hybrid Search + Reranker」コンボ:** AWS はほぼコード不要で最良箇所をトップに押し上げる 2 つの武器を用意:
1. **Hybrid Search:** vector（曖昧な意味）だけでなく **keyword**（厳密一致）も併用。両者が補完し、最も正確な文書を自動で上げる。Bedrock Knowledge Bases では **設定トグル** だけ — 楽。
2. **Reranker Model:** 「審査員」役。20 箇所を取得後、reranker（例 Bedrock の Cohere Rerank）に全部渡すと、再採点して並べ替え、最良箇所を 1 位に。**built-in** で学習不要。

### 🧩 選択肢の分解
- **A — 正解。** pure vector から hybrid search への切替は RAG 精度を上げる最速・最も効果的な手段。Knowledge Bases（OpenSearch Serverless backend）での設定は **built-in 機能の有効化** だけ、アルゴリズム不要、低工数。
- **C — 正解。** Bedrock reranker は「良い結果が下に埋もれる」問題に **ぴったり**。自動で reorder。managed なので *minimal operational overhead* を完全に満たす — API を呼ぶだけ。
- **B — 不正解（手間が多すぎ）。** Learning to Rank は OSS プラグイン: **click データを自前収集**、スコアモデルを **自前学習**、**自前保守** が必要。overhead が膨大、要件違反。
- **D — 不正解（車輪の再発明）。** pgvector は良いが、開発者に **「類似度スコアアルゴリズムを自作」** させるのは苦行: 数式を手書き、インフラを自前管理 — "minimal overhead" の真逆。
- **E — 不正解（過剰設計）。** Bedrock に reranker が built-in なのに、SageMaker JumpStart + Kendra を継ぎ「独自アルゴリズム」を作るのは重すぎ。

### ✅ 結論: **A + C。**

### 💡 試験のコツ
1. **「Improve ranking / most relevant lower / better retrieval」** → AWS の無敵ペアは **Hybrid Search**（vector + keyword）と **Reranker Models**（再採点 + 並べ替え）。
2. **致命的キーワード「MINIMAL overhead」:** *Create custom algorithm*、*Train custom model*、*Learning to Rank plugin* を含む選択肢は即消し。常に Bedrock の **built-in/managed** 機能を優先。

</details>

---

## 問 2 — リアルタイムかつ resilient な Knowledge Base 同期

> 古典的アーキテクチャパターン: **リアルタイムイベントを resilient に処理**（event-driven & resilient）。多数のファイルが同時に変わっても **クラッシュさせず**、AI に即時更新させるには?

ある病院が、社内手順文書に基づき回答する AI アシスタントを Amazon Bedrock Knowledge Bases（ソースは Amazon S3）で運用。新規文書は **できるだけ早く** 回答に反映、削除文書は **できるだけ早く** 除外したい。解は **scalable・event-driven・resilient（耐障害）** であること。

正しい解は?

- **A.** EventBridge Scheduler で 5 分ごとに Lambda を実行、S3 の変更を検知して追加/削除 API を呼ぶ。
- **B.** S3 Event Notifications が作成/削除時に Lambda を直接起動; Lambda が `IngestKnowledgeBaseDocuments` / `DeleteKnowledgeBaseDocuments` を呼ぶ。
- **C.** S3 Event Notifications が作成/削除イベントを **Amazon SQS** キューへ送る; Lambda がキューを poll して `IngestKnowledgeBaseDocuments` / `DeleteKnowledgeBaseDocuments` を呼ぶ。
- **D.** EventBridge Scheduler で 5 分ごとに Lambda を実行、`StartIngestionJob` で全体を再同期。

<details>
<summary>📖 分析と解答を見る</summary>

### 🎯 この問題が試す concept
**event-driven + SQS による buffering（緩衝）** で、リアルタイムかつ負荷スパイクに強い構成。Domain 2。

### 🔍 核心分析 — 「超高速更新だがクラッシュ厳禁」
3 つのキーワードが選択肢を削る:
1. **「できるだけ早く」:** リアルタイム必須、ファイルが来たら即更新 → **Scheduled/Cron を排除**。
2. **「event-driven」:** ファイル変更（イベント）が自動でシステムを起動。
3. **「resilient」— 最重要キーワード:** 病院が 1 秒で 1,000 ファイルをアップすると Bedrock が溢れて **throttling**。resilient のためには、その 1,000 を貯めて少しずつ Bedrock に流す **「漏斗/緩衝」** が必須。

**特効薬 — Amazon SQS:** SQS キューがその漏斗。S3 の全イベントを受け、安全に保持してデータを落とさず、Lambda の Bedrock 更新が失敗しても **自動 retry**。Lambda がメッセージを 1 件ずつ取り、安定速度で処理、過負荷にならない。

### 🧩 選択肢の分解
- **C — 正解。** `S3 → SQS → Lambda → Bedrock API` は AWS のベストプラクティス: S3 Event Notifications が「event-driven」と「リアルタイム」を、SQS が「resilient」（負荷緩衝 + 自動 retry）を提供。スパイクでも scale 可能でクラッシュしない。
- **A — 不正解。** 5 分ごとは「できるだけ早く」に反し event-driven でもない。Lambda に **「S3 の変更を自前で検知」**（旧/新ファイル比較）させるのはリソース浪費で非効率。
- **B — 不正解（resilient 不足で死亡）。** `S3 → Lambda` 直接は速く event-driven だが **buffer が無い**。スパイク時 Bedrock が throttling → Lambda 失敗 → 失敗イベントを保持するキューが無く、そのファイルは **永遠に更新されない**（データ消失）。
- **D — 不正解。** 5 分で遅く、API も誤り: `StartIngestionJob` は **S3 全体を再スキャン・再同期**、変更ファイルだけの incremental（`IngestKnowledgeBaseDocuments`）よりはるかに遅く高い。

### ✅ 結論: **C。**

### 💡 試験のコツ
1. **「As soon as possible / near real-time」+「Resilient」/「Traffic spikes」:** 正解にはほぼ必ず **Amazon SQS** が緩衝として登場（decoupling/buffering）。
2. **Direct invocation（S3 → Lambda）:** "resilient" や "traffic spikes" を強調していたら即除外。
3. **Scheduler / Cron job:** "as soon as possible" や "event-driven" があれば即消し。スケジューリングは **リアルタイムの敵**。

</details>

---

## 問 3 — 最小の手間で画像/動画を分析

> AI に **「目」** を持たせる課題: 写真と動画を分析しトレンドを dashboard にまとめる — IT チームがラベル付けやモデル学習で **「身を粉にしない」** で。

あるスポーツメディア企業が、公開試合の **動画と写真** を分析して戦術要素やトレンドを把握し、抽出情報を保存して要約 dashboard を作りたい。要件は **最小の operational overhead**。

正しい解は?

- **A.** EventBridge → Lambda で QuickSight Q を使い動画/写真を分析、S3 に保存、QuickSight dashboard。
- **B.** **Step Functions** で **Amazon Bedrock の multimodal FMs** を使い動画/写真を処理、S3 に保存、**QuickSight** で可視化。
- **C.** Rekognition Custom Labels で独自モデルを学習、DynamoDB に保存、Managed Grafana + 独自プラグインで dashboard。
- **D.** Claude でテキスト記述を分析、Stable Diffusion で画像を「分析」、OpenSearch に保存、Grafana で dashboard。

<details>
<summary>📖 分析と解答を見る</summary>

### 🎯 この問題が試す concept
**built-in の multimodal Foundation Model** で画像/動画内容を理解（自前学習なし）+ managed dashboard。Domain 1 & 2。

### 🔍 核心分析 — 「目を持つ AI」
入力は **動画と写真**（visual data）。AI が「見る」には **multimodal** な頭脳が必要。さらにキラーキーワード **LEAST operational overhead** — 出来合いのサービスを選び、**自前学習なし**、サーバ管理なし、UI コードも書かない。

**特効薬 — Bedrock Multimodal + QuickSight コンボ:**
1. **Multimodal FMs**（Claude、Amazon Nova Pro）: 文字も読め、画像/動画も「見える」。追加学習不要 — 選手がどの戦術を走っているか自分で理解。
2. **Amazon QuickSight:** serverless のドラッグ&ドロップ BI/dashboard ツール — トレンド報告には楽。

### 🧩 選択肢の分解
- **B — 正解。** Bedrock multimodal FM（出来合い）が動画を直接「見る」、ラベル付け/学習不要; Step Functions が抽出を編成（ベストプラクティス）; QuickSight が S3 データ（Athena 経由）から複雑なソフト/プラグイン無しで dashboard。
- **A — 不正解（QuickSight Q の誤解）。** QuickSight **Q** は **表/数値データ** への自然言語 Q&A（例「今月の売上は?」）。`.mp4`/`.jpg` を「見て」分析することは **できない**。
- **C — 不正解（運用負荷が高い）。** Rekognition Custom Labels = 数千枚を自前アップ、**bounding box を描き**、ラベル付け、**学習**; さらに Grafana 用 **独自プラグイン** を自作。工数膨大、"least overhead" 違反。
- **D — 不正解（重大な道具違い）。** **Stable Diffusion は画像生成モデル（text-to-image）** — 新規画像を *描く* もので、既存画像を *分析/認識* しない。OpenSearch クラスタの自前管理も工数増。

### ✅ 結論: **B。**

### 💡 試験のコツ
1. **「Analyze videos and photos / image understanding / no custom training」** + LEAST overhead → **Bedrock の Multimodal FMs**（Claude、Nova）。
2. **Rekognition Custom Labels:** 「自社のラベル付き画像セットがある + 独自ロゴ/製品を認識」と明記された時だけ。一般的な物体認識は出来合いサービス。
3. **「Dashboard / BI」を楽に（serverless/managed）** → **Amazon QuickSight** が既定解。

</details>

---

## 問 4 — モデル置換の評価ワークフローを順序付け

> **順序付け（ordering）** 問題、非常に現実的なシナリオ: 顧客対応中の古い AI を新しい AI に **「戦の最中に大将を替える」** — システムを壊さずに行うには?

ある企業が本番の翻訳モデルを新モデルに置き換えたい。**新モデルがより良いと証明された場合のみ**。プロセスは **逐次的（sequential）** で、各ステップは次へ進む前にレビュー・承認される。次のステップを **順序付け** せよ:

- 結果を分析し、包括的な評価レポートを作成する。
- 新モデルと現行本番モデルを比較する A/B testing を実施する。
- 多様なシナリオと edge cases を含む test dataset を作成する。
- 評価 metric（関連度・事実正確性・流暢性）を定義する。
- AWS Step Functions で automated quality gates（自動品質ゲート）を実装する。

<details>
<summary>📖 分析と解答を見る</summary>

### 🎯 この問題が試す concept
体系的な evaluation プロセス — **科学的かつ逐次的**、まず物差しを決め、それから測る。Domain 5。

### 🔍 核心分析 — 「人材採用試験を行う」
新 AI への入れ替えは超リスク; 「賢そう」で即投入はできない。AWS は試験運営のような原則を要求: **採点ルールを作る → 試験問題を作る → 本番試験 → 自動ゲートで採点 → 最終判断**。

### ✅ 正しい順序（と理由）
1. **Define evaluation metrics** — 何より先に **「何が良いか」** を知る（正確度何 %、流暢さ）。後続全ステップの物差し。
2. **Create test dataset** — 採点ルールが揃ったら **「試験問題」** を用意: 難問・edge cases 入りのデータを両モデル共通で。
3. **Conduct A/B testing** — **「リングに上げる」**: 旧 (A)・新 (B) 両モデルにステップ 2 の問題を解かせ、ステップ 1 の metric で測定。
4. **Implement automated quality gates (Step Functions)** — **「機械の審査員」**: Step Functions が新モデルが閾値を超えたか自動確認し、**人の承認（approval）を強制** してから先へ。
5. **Analyze results & generate report** — 全ゲート通過後、**証拠を集約** したレポートで経営が「置換 OK」を押す根拠に。

### 💡 試験のコツ
評価の workflow/ordering 問題では呪文: **Metrics → Dataset → Test → Gate/Threshold → Report**。評価 metric を定義し終える前にデータを test へ入れることは絶対にない。

</details>

---

## 問 5 — 全呼び出しに Guardrail を強制

> **エンタープライズ AI ガバナンス** の問題: 全開発者が AI 呼び出し時に必ず safety filter（Guardrails）を通るよう **「強制」** するには、各人を **見張らず** にどうするか?

ある製造コングロマリットが AI ガバナンスポリシーを展開: FM への **すべての** `InvokeModel` と `Converse` 呼び出しに Bedrock Guardrails を **必須** 適用。各チームの開発者が code で guardrail を「付け忘れる」可能性。解は **MOST operationally efficient** であること。

正しい解は?

- **A.** IAM policy に `bedrock:GuardrailIdentifier` **と** `bedrock:PromptRouterArn` の両 condition key を設定、prompt router 検証を要求。
- **B.** Lambda を proxy として書き、Bedrock へ転送する前に guardrail を検証・強制; 全対話を Lambda 経由に強制。
- **C.** guardrail ID を Parameter Store に保存; Lambda が Bedrock 呼び出し前に毎回 ID を取得。
- **D.** IAM policy に condition key **`bedrock:GuardrailIdentifier`** を設定、FM にアクセスする全 IAM role に適用。

<details>
<summary>📖 分析と解答を見る</summary>

### 🎯 この問題が試す concept
**IAM Policy + Condition Key** で **インフラ層** に強制 — 他人の code を検査する code を書かない。Domain 3。

### 🔍 核心分析 — 「ルール遵守の強制」
開発者は code に guardrail ID を入れ「忘れ」たり意図的に省ける。ID が無いと AI は **safety filter 無し**（暴言防止、PII 漏洩防止）で応答。

**特効薬 — IAM の関所:** 最も賢く楽なガバナンスは、code を検査する code を書くことではなく **IAM Policy + Condition Key**。AWS の門にルールを置く: *「Bedrock API を呼ぶ際、リクエストに `bedrock:GuardrailIdentifier` が無ければ、私（AWS）が即 Access Denied で弾く!」*。**無料・built-in・保守ゼロ**。

### 🧩 選択肢の分解
- **D — 正解（ゴールドスタンダード）。** IAM policy で `bedrock:GuardrailIdentifier` condition key を使うのが正攻法で最も効率的。guardrail ID を含まない呼び出しは IAM が **インフラ層で自動ブロック**。custom code 不要 → 運用効率最高。
- **A — 不正解（蛇足）。** 正しい condition key はあるが `bedrock:PromptRouterArn` を強制。Prompt Router は **prompt 経路制御** で、guardrail 強制とは **無関係**; 無関係条件は policy を無意味に複雑化。
- **B — 不正解（自前のボトルネック）。** Lambda proxy を間に挟むと **クラッシュ・過負荷・latency 増** を全クエリに招き、code を常に保守 → "operationally efficient" に重大違反。
- **C — 不正解（遅く高コスト）。** B 同様 Lambda で減点; さらに **毎回** Parameter Store に ID を取りに行く → 無駄な遅延 + Parameter Store API を数千回呼ぶコスト。

### ✅ 結論: **D。**

### 💡 試験のコツ
1. **「Enforce Bedrock Guardrails」+「MOST operationally efficient」** → 答えは常に **IAM Policy + condition key `bedrock:GuardrailIdentifier`**。
2. **custom proxy を避ける:** AWS が Access Control / Governance を求めたら常に **IAM**。権限チェックのため **Lambda を proxy** に挟む選択肢は消す — serverless ガバナンスの思想に反する悪設計。

</details>

---

## 問 6 — 特定フレーズで生成を停止

> **inference parameters** の問題 — 具体的には AI の **「ブレーキペダル」**: 事前定義のフレーズを書いた瞬間に **即停止** させるには?

ある開発者が Anthropic Claude on Bedrock で仮想アシスタントを構築。応答中に **特定フレーズが生成された直後に出力を止めたい**。

正しい解は?

- **A.** user prompt に「このフレーズで止まれ」を追記。
- **B.** 推論呼び出しで **stop sequences（停止シーケンス）** パラメータにトリガーフレーズを指定。
- **C.** **top-k** パラメータで token の多様性を制御。
- **D.** **temperature** パラメータでフレーズ出現の可能性を制御。

<details>
<summary>📖 分析と解答を見る</summary>

### 🎯 この問題が試す concept
各 inference parameter の機能理解; **stop sequences** が信頼できる停止機構と認識。Domain 2。

### 🔍 核心分析 — 「ブレーキペダル」
「Conclusion:」を書いた瞬間に止めたい。
- **素朴な方法:** prompt で頼む — 運転手に口頭で止まれと言うようなもので、時に **「行き過ぎ」**。LLM は特に高創造性時に「口を滑らせ」やすい。
- **特効薬 — 自動遮断器（stop sequences）:** Bedrock API には `stop_sequences = ["Conclusion:"]`。これは **サーバの自動ブレーキ**: その文字列を出そうとした瞬間に **即接続を切り**、AI がそれ以上一語も言わないことを 100% 保証。

### 🧩 選択肢の分解
- **B — 正解。** stop sequences はこの目的のために作られた。単語/文字の配列（例 `["\n\nHuman:", "Conclusion:"]`）を渡すと、その文字列を生成しかけた時点で token 生成が **即停止**。100% 信頼できる技術的制御。
- **A — 不正解（AI の気分次第）。** prompt の指示はモデルの服従意思に依存。LLM は完全な if-else 機械ではなく、途中で指示を無視して生成を続け得る → コア機能には信頼性不足。
- **C — 不正解（Top-k の誤解）。** Top-k は次 token に **考慮する語彙数** を制限（例 上位 50 語から 1 つ）; 多様性に影響するが **「停止」能力は無い**。
- **D — 不正解（Temperature の誤解）。** Temperature は **ランダム性/創造性** を制御（高=華やか、低=定型）; 生成を中断/停止する機能は無い。

### ✅ 結論: **B。**

### 💡 試験のコツ（4 つの古典パラメータ）
1. 特定の文字/語で **止める** → **stop sequences**。
2. **hallucination** を減らし、真面目で安定した回答 → **temperature を 0 に下げる**。
3. AI を **創造的** に、詩/物語 → **temperature を 1 近くに上げる**。
4. AI の **語彙** を制御（変な語を避ける）→ **Top-P / Top-K** を調整。

</details>

---

## 問 7 — LLM エンドポイントのリソース利用最適化（2 つ選択）

> **LLM デプロイのコスト/ハードウェア最適化** 問題: **50 席のバスに 5 人しか乗せない** ような大きな無駄! どうバスを満席にするか?

fine-tuned LLM が SageMaker AI endpoint に continuous batching（DJL ライブラリ）でデプロイ済み。各 EC2 は **8 GPU**。本番でインスタンスが多すぎ → コスト増。ログ: (1) 実際の I/O sequence length が設定より **10 倍小さい**、各インスタンスの concurrency が低い; (2) 重み + activations は **4 GPU** で収まる。

リソース利用を改善する 2 つは?（2 つ選択）

- **A.** SageMaker インスタンス数を増やし、リクエストを均等に分散。
- **B.** model の **maximum sequence length を下げ**、GPU あたりの rolling batch size を上げる。
- **C.** speculative decoding を有効化し、リクエストごとの latency を下げる。
- **D.** **tensor parallelism degree = 4** で、各インスタンスに **2 つの replicas** をデプロイ。
- **E.** tensor parallelism degree = 8 で model を 8 GPU 全体に分割。

<details>
<summary>📖 分析と解答を見る</summary>

### 🎯 この問題が試す concept
continuous batching での GPU VRAM 最適化と tensor parallelism の割り算。Domain 4。

### 🔍 核心分析 — 無駄の 2 つの病気
**病気 1 — 「席だけ予約して誰も座らない」（Sequence Length）:** LLM サーバ（DJL/vLLM）では **KV Cache** の VRAM が *Max Sequence Length* でロックされる。1 万 token 設定 → GPU が 1 万分の VRAM を予約。でも実際は 1,000 → 余剰 VRAM が **空き**、本来は同時に多くの客を捌ける（batch size/concurrency 増）。

**病気 2 — 「大きな窯で小さなケーキ」（Tensor Parallelism）:** 8 GPU の高級 EC2 を借りたが model は 4 GPU で足りる。既定だと model は 4 GPU で動き、**残り 4 GPU が遊ぶ**。

**治療:** (1) 上限を実態に合わせ下げる → VRAM 解放 → batch size 増（客を詰める）。(2) 8 GPU を使い切るため model を **複製**: 8 GPU ÷ 4 GPU/model = **2 replicas** を同一マシンに → スループット倍増。

### 🧩 選択肢の分解
- **B — 正解。** DJL はメモリを厳格管理。実利用が遥かに小さい（10 倍）なら Max Sequence Length を下げるのが的確: 「席予約」から VRAM が解放され、より多くの同時ストリームを処理（higher rolling batch size）。
- **D — 正解。** model が 4 GPU に収まる → **TP = 4**; マシンは 8 GPU → DJL が **2 replicas** を自動生成（4×2=8）。1 台で 2 台分のスループット。
- **A — 不正解（金を燃やす）。** 「インスタンス増設」はまさに避けたいコスト増の原因; 現マシン内の無駄を解決しない。
- **C — 不正解（病気違い）。** speculative decoding は **token 生成を速く（latency 減）** するが VRAM 節約や利用率向上はせず、draft model を並走させ **VRAM を余計に消費**。
- **E — 不正解（無駄な引き伸ばし）。** TP = 8 は 4 GPU で足りる model を 8 GPU に分散; マシンは **1 replica のまま**、スループットは増えず、8 GPU の通信 overhead でむしろ遅くなり得る → 2 つ目の replica を作る機会を浪費。

### ✅ 結論: **B + D。**

### 💡 試験のコツ
1. **Tensor Parallelism:** 常に割り算 — EC2 に **X** GPU、model が **Y** GPU に収まる → 最良は **TP = Y**、replica 数 = **X / Y**。必要なければ TP = X に広げない。
2. **Continuous Batching / VRAM / Sequence Length** は **「シーソー」**: Max Sequence Length を下げる → VRAM 余る → Max Batch Size を上げる → 同時により多くの客を捌く。

</details>

---

## 問 8 — Web エディタへのリアルタイム提案ストリーミング

> おなじみの UX: 10 秒スピナーを見せる代わりに、AI が **「一文字ずつタイプ（streaming）」** で画面に即表示するには?

ある法律事務所が Web エディタを構築: 弁護士が「分析」をクリックすると、システムは **即座に** 条項修正提案を（タイプライター効果で）インタフェースへ stream 開始。Bedrock FM を使用。要件は **最小の operational overhead**。

正しいアーキテクチャは?

- **A.** SQS が記事を受信 → Step Functions 処理 → Lambda → DynamoDB → API Gateway WebSocket で stream。
- **B.** **API Gateway WebSocket** に **Lambda** を連携; Lambda が metadata を読み model を routing、**Bedrock Prompt Management** で style ルールを適用、**Bedrock streaming API** でリアルタイム返却。
- **C.** API Gateway **REST API** + Lambda function URLs、chunked transfer encoding で stream。
- **D.** Application Load Balancer + ECS（custom container）で routing ロジック + Bedrock streaming、WebSocket で Web へ。

<details>
<summary>📖 分析と解答を見る</summary>

### 🎯 この問題が試す concept
**Web UI へのリアルタイム response streaming** 構成、フル serverless。Domain 2。

### 🔍 核心分析 — 「AI が生でタイプ」
通常 HTTP（リクエスト送信 → 全体待ち → 一塊返却）では UX が悪い。ChatGPT のような「タイプライター」効果には、Web が **「開きっぱなしのパイプ」** をサーバと維持: token が生成され次第、サーバが画面へ即押し出す。

**特効薬 — WebSocket + Bedrock Streaming API:**
1. **WebSocket（双方向パイプ）:** API Gateway の WebSocket API がブラウザとサーバを継続接続、途切れない。
2. **Bedrock Streaming API:** plain `InvokeModel` でなく `InvokeModelWithResponseStream` — AI が出す token を即 WebSocket パイプへ → 弁護士の画面へ直送。速く滑らか、遅延なし。

### 🧩 選択肢の分解
- **B — 正解（ゴールドスタンダード）。** API Gateway WebSocket + Lambda + Bedrock Streaming API はリアルタイム chatbot/AI アシスタントのベストプラクティス。全 serverless → "最小の operational overhead"。Lambda は超高速の交通整理役で metadata を読み適切な prompt へ routing; Prompt Management が style を統制。
- **A — 不正解（キューがリアルタイムを窒息）。** 先頭に **SQS** は致命的: SQS は **非同期（async）**、メッセージ取得に「待ち（poll）」が要る → 遅延発生、クリック時の「即時フィードバック」を完全に破壊。
- **C — 不正解（プロトコル違い）。** REST API は **短命・同期** 接続向け（1 リクエスト-1 レスポンスで閉じる）; WebSocket のように接続を保持して chunk を滑らかに stream できない。数十秒の AI タスクは **timeout** しやすい。
- **D — 不正解（自前の足枷）。** 小さな routing ロジックのため ECS クラスタを立て、Dockerfile を書き、container を管理するのは **「鶏を割くに牛刀」** — overhead 高、要件違反。

### ✅ 結論: **B。**

### 💡 試験のコツ
1. **「Real-time feedback / typewriter effect / streaming to web UI」** → 勝ちペアは **API Gateway WebSocket + Bedrock Streaming API**。
2. **リアルタイム UI から SQS を外す:** SQS は background 処理に最適だが、ユーザーがクリックして画面で結果を待つ場面では絶対不可。
3. **Serverless > Containers:** 「LEAST operational overhead」なら Lambda/API Gateway を優先、自前 EC2/ECS を要する選択肢は消す。

</details>

---

## 問 9 — prompt ガバナンス + 長期コンプラ logging（2 つ選択）

> 金融の生命線 2 つ: **人の承認ワークフロー管理（governance）** と **改ざん不可のログを監査向けに保存（compliance/audit）**。

ある保険会社が複数事業部の CS アシスタントに Bedrock を使用。必要: (1) prompt templates を **approval workflow** で統制; (2) **全 model invocation を logging** し **7 年保持** して法規制遵守。要件は **最小の operational overhead**。

正しい 2 つは?（2 つ選択）

- **A.** **Amazon Bedrock Prompt Management** を多段 approval workflow で使う。
- **B.** prompt を DynamoDB に保存、IAM + item-level 権限を自作して承認を制御。
- **C.** EventBridge で invocation イベントを取得 → CloudWatch Logs → S3 へ export、S3 Object Lock を 7 年で有効化。
- **D.** 全 Bedrock API の CloudTrail data events を有効化、CloudTrail Lake に 7 年保持で配信。
- **E.** **Bedrock model invocation logging** を S3 宛に有効化、**S3 Object Lock（compliance mode）を 7 年** で有効化、事業部ごとに prefix を作る。

<details>
<summary>📖 分析と解答を見る</summary>

### 🎯 この問題が試す concept
2 ピース: (1) managed 機能による prompt ガバナンス; (2) chat **内容** を 7 年不変保存。Domain 3。

### 🔍 核心分析 — 明確な 2 つの半分
**半分 1 — prompt の管理と承認:** 大企業では開発者が勝手に prompt を直し顧客に出せない。下書き → 上司承認 → 本番 のワークフローが必須。AWS にこの「純正」機能がある。

**半分 2 — 金融監査向け logging:** 監査人は「誰が AI をいつ呼んだか」だけでなく、**顧客が何をチャットし AI が何を答えたか（full payload）** を正確に知りたい。さらにログは **7 年「凍結」**、誰も（IT 部長も、Root アカウントも）削除/改変できないこと。

### 🧩 選択肢の分解
- **A — 正解（半分 1）。** Amazon Bedrock Prompt Management は built-in: 作成 → parameterize → version → **approval workflow** の UI とフローを提供。出来合いで工数最小。
- **E — 正解（半分 2）。** **Model invocation logging** が prompt + response の **全体** を S3 に直送。**S3 Object Lock の Compliance mode** は最高保護（WORM — Write Once Read Many）: ロック後は **AWS のどんな権限でも** 7 年の期限前に削除/上書き不可 — 金融規制（SEC/FINRA）の必須標準。
- **B — 不正解（車輪の再発明）。** DynamoDB に prompt 保存は可だが、**管理 Web と approval workflow ロジックを自作** → A があるのに膨大で不要な overhead。
- **C — 不正解（迂回でデータ漏れやすい）。** EventBridge → CloudWatch → S3 export は嵩張り、破断点が多く、長いチャットの **full payload を取り切れない** → "minimal overhead" 違反。
- **D — 不正解（CloudTrail の古典的罠）。** CloudTrail は玄関の防犯カメラ: **metadata のみ** 記録（User John が 10:00 に `InvokeModel`）。**prompt/response の内容は絶対に保存しない**。金融監査では従業員が AI に何を聞いたか不明では **無価値**。

### ✅ 結論: **A + E。**

### 💡 試験のコツ
1. **「Prompt versions / approval workflows」** → **Amazon Bedrock Prompt Management**。
2. **「Regulatory compliance / cannot be deleted / N-year retention」** → 究極の武器は **Amazon S3 Object Lock（Compliance mode）**。
3. **AI ログを明確に区別:** **誰が API を呼んだか** → CloudTrail; **chat 内容（prompt/response payload）** → **Bedrock Model Invocation Logging**。**内容** の監査を求められたら CloudTrail を選ばない。

</details>

---

## 問 10 — Python agent を AgentCore Runtime へデプロイ（2 つ選択）

> AI Agent の Python code が出来たら、**「大工仕事」**（サーバ構築、Docker パッケージ、セキュリティ自前）をせずに Bedrock 上の正式サービスにするには?

ある fintech 企業が **既存の Python agent code** を **Amazon Bedrock AgentCore Runtime** へデプロイし、**インフラ管理を最小化** したい。agent は高速ルックアップ（1 秒未満）と長尺レポート（数分の streaming）の両方を処理。解は **自動で** HTTP server、endpoint routing、health monitoring を管理すること。

**overhead 最小** の 2 つは?（2 つ選択）

- **A.** FastAPI server を自前構築し `/invocations`、`/ping` を設定、container も自前編成。
- **B.** **AgentCore SDK** の **`@app.entrypoint`** decorator で server + endpoint を自動処理。
- **C.** ECS Fargate に AgentCore SDK を動かす custom container でデプロイ。
- **D.** SageMaker AI real-time endpoint に custom inference container でデプロイ。
- **E.** **AgentCore starter toolkit** で packaging、containerization、deploy を自動化。

<details>
<summary>📖 分析と解答を見る</summary>

### 🎯 この問題が試す concept
最小インフラで code を Runtime に載せる AgentCore のツール群。Domain 2。

### 🔍 核心分析 — 「code は少しなのにサーバ構築が多い」病
Bedrock が Python ファイルと話すには、それを **HTTP server** にし、port を開け、health 用 `/ping` とリクエスト用 `/invocations` を持たせる必要。AI 開発者にこれを書かせ、Docker をパッケージし、ECR に push させると **消耗** する。

**特効薬 — AgentCore SDK & Starter Toolkit** が 2 階層で開発者を救う:
1. **Code 階層（`@app.entrypoint`）:** 冗長な FastAPI を書く代わり、Python 関数の上に **1 行** `@app.entrypoint` を置くだけ。AWS が正式な server に自動ラップ、高速応答も長尺 streaming も対応。
2. **デプロイ階層（starter toolkit）:** 面倒な `docker build`/`docker push` の代わり、toolkit が **code を自動 package、container 化、deploy** を Bedrock へ。

### 🧩 選択肢の分解
- **B — 正解（code 階層）。** `@app.entrypoint` decorator は「自動で HTTP server 管理」を満たす秘密兵器、web framework を書く必要を排除。高速 JSON（1 秒未満）も streaming（数分）も滑らかに対応。
- **E — 正解（インフラ階層）。** starter toolkit が退屈なインフラ（Dockerfile、ECR、AgentRuntime）を担当 → 最短・最小工数（minimal overhead）。
- **A — 不正解（車輪の再発明）。** 自前 FastAPI は port 管理、`/ping`・`/invocations` ロジック、streaming 処理を自前 → overhead 急増。
- **C, D — 不正解（プラットフォーム違い）。** **ECS Fargate**（C）や **SageMaker custom container**（D）は Docker ライフサイクル、Task 定義、Load Balancer/Auto Scaling の自前管理を要求。指定された宛先は **AgentCore Runtime** — 迂回は要件違反かつ手間過大。

### ✅ 結論: **B + E。**

### 💡 試験のコツ
1. **AgentCore Runtime** + 「Minimal overhead / No manual server config」 → 2 キーワードを探す: **`@app.entrypoint`**（code）と **AgentCore starter toolkit**（deploy）。
2. **最速・最楽** を問われたら、web framework 自作（FastAPI/Flask）や custom container 自作の選択肢は消す。

</details>

---

## 問 11 — 生成コンテンツの source lineage（2 つ選択）

> GenAI の **data lineage（来歴）** 問題: AI がコンテンツを生成した時、レビュー担当が **どのソース** から知識を得たか分かり、正確性を検証するには?

ある教育出版社が Bedrock で練習問題生成システムを構築、**curated（厳選）** と **scraped（Web 収集）** の混在データを使用。レビュー担当は **どのソースか（source lineage）** を知って **信頼性を検証** する必要。要件は **最小の operational overhead**。

正しい 2 つは?（2 つ選択）

- **A.** Bedrock invocation logging を有効化し、ログとソースを相関させる仕組みを自作。
- **B.** FM の **output にソースデータの metadata を tag 付け** する。
- **C.** データセット（curated + scraped）を **AWS Glue Data Catalog** に **登録** する。
- **D.** SageMaker Clarify でモデル予測を説明。
- **E.** CloudTrail でレビュー担当の承認操作を log。

<details>
<summary>📖 分析と解答を見る</summary>

### 🎯 この問題が試す concept
Source lineage = output に **「ソースラベル」** を貼る + 中央の **「ソース登録簿」** を持つ。Domain 1 & 3。

### 🔍 核心分析 — 「産地ラベル」問題
AI システムをパン工場と想像、原料は 2 か所: **認証農場**（curated）と **フリーマーケット**（scraped）。レビュー担当（食品安全検査官）が完成品（AI 生成の問題）を見て聞く: *「これどの小麦?」*。工場を掘り返さず答えるには 2 つ要る:
1. **ケーキにラベル:** 窯から出た瞬間、ロット ID の **metadata tag** を貼る。
2. **ソース登録簿:** ID 1 = 農場 A、ID 2 = 市場 B を列挙。検査官は tag を見て登録簿で照合、完了。

### 🧩 選択肢の分解
- **B — 正解（ラベル貼り）。** AI が output を出すたび **元文書の metadata** を自動埋め込むようプログラム。レビュー担当は即 `Source: Wikipedia_Biology_p42` の tag を見る。直接的・自動・軽量。
- **C — 正解（登録簿）。** AWS Glue Data Catalog が中央「登録簿」: 全ソース（curated + scraped）を index・管理。B の tag を見て Data Catalog で信頼性を照合。**managed service** → "least overhead" を満たす。
- **A — 不正解（手作業・複雑）。** invocation logging は「prompt X → output Y」だけ記録; **どの PDF から得たかは分からない**。相関させるには非常に複雑な照合システムを自作 → 工数大。
- **D — 不正解（道具違い）。** SageMaker Clarify は **bias** 検出と古典 ML の **explainability**（なぜこのローンを承認したか）専用。GenAI の **source lineage** は追跡しない。
- **E — 不正解（的外れ）。** CloudTrail は AWS 上の **人の操作**（API calls）のみ: 「A さんが 3 時に承認」。問題のデータソースには無知。

### ✅ 結論: **B + C。**

### 💡 試験のコツ
1. **「Source lineage / data origin / traceability」** → ペア **Metadata Tagging**（output へ）+ **AWS Glue Data Catalog**（中央 metadata ストア）。
2. **Explainability と Lineage を混同しない:** Clarify はモデルが **どう** 考えるか; Data Catalog は **どこから** 入力データを得たか。

</details>

---

## 問 12 — デプロイ後に RAG が「静かに故障」

> RAG をトラブルシュートする **「AI 探偵」**: 赤いエラーは無く、全て完璧に見えるのに結果は **大外れ**!

ある金融サービス企業が RAG を運用: embedding model は Bedrock、vector store は OpenSearch、embedding + 検索ロジックは Lambda。**Lambda の code 更新後**、以前は良く答えていた質問にも「関連情報が見つかりません」を返し始めた。CloudWatch は **エラー無し**、X-Ray は **FM 呼び出し成功** を確認、OpenSearch は健全、latency も正常。

原因は?

- **A.** 更新時に OpenSearch の文書 vector が削除され、再 index されていない。
- **B.** Lambda の IAM role に `bedrock:InvokeModel` 権限が欠落。
- **C.** FM の temperature パラメータが上げられた。
- **D.** 更新後の Lambda が **異なるバージョンの embedding model** を使っている。

<details>
<summary>📖 分析と解答を見る</summary>

### 🎯 この問題が試す concept
**Embedding drift / vector space mismatch** — と「現場の痕跡」を読んで消去する力。Domain 5。

### 🔍 核心分析 — 「日本語とフランス語」問題
当初、全文書は「**日本語**」（embedding model V1）に翻訳され OpenSearch に保存。今日、開発者が Lambda の code を更新し、うっかり「**フランス語**」モデル（V2）に切替。ユーザーが質問すると、Lambda は質問を「フランス語」に訳し、「日本語」だらけのライブラリで類似検索。

技術的には **何もエラーしない**: Lambda 正常、OpenSearch 健全、エラーログ無し。だがフランス語と日本語は **数学的座標系がずれる** → OpenSearch が 0 件 → 「関連情報が見つかりません」。これが **Embedding Drift**。

### 🧩 選択肢の分解
- **D — 正解。** embedding version の切替（例 `titan-embed-text-v1` → `v2`）は **全く別の vector space** を作る。新 code は query を新座標に変え、旧モデルで埋め込んだ文書と一致しない。故障が **「静か」** なのでエラーログが無い（CloudWatch クリーンの手掛かりと一致）。
- **A — 不正解。** **Lambda の code 更新** が OpenSearch のデータを自動消去はできない; DB が空なら通常は空クエリ警告が出る。問題の核心は code 更新フロー。
- **B — 不正解（前提を否定）。** 問題文に **「X-Ray が FM 呼び出し成功を確認」** と明記。IAM 権限欠如なら X-Ray/CloudWatch は **`AccessDeniedException` で溢れ**、app は crash; 「見つかりません」と穏やかに返さない。
- **C — 不正解（Temperature の誤解）。** Temperature は **step 2（Generation）** のみに影響。だが「関連情報が見つかりません」は **step 1（Retrieval）** の失敗の証; LLM が元文書を見つけられず諦めた。Temperature は OpenSearch の検索能力と無関係。

### ✅ 結論: **D。**

### 💡 試験のコツ
1. **RAG の Silent failure:** *システム正常 → code 更新 → 全て「緑」（エラー無し、latency 良）→ だが検索が空/大外れ* → ほぼ確実に **embedding model version の変更**。
2. **黄金律:** model A の vector space は model B と **決して** 互換でない（同一ベンダの v1 vs v2 でも）。embedding model を替えたら DB 全体を **必ず re-index**。

</details>

---

## 問 13 — Knowledge Base へのドキュメント取込を監視

> 運用エンジニアの仕事: 1,000 ファイルを Bedrock に読ませる時、**どれが成功しどれが失敗したか**（パスワード暗号化、サイズ超過、PDF 破損）をどう知るか?

ある企業が、複数 S3 bucket から取込む Bedrock Knowledge Base を使う Q&A アプリを運用。**ingestion（取込）プロセスを監視** し、**どの文書が処理に失敗したか** を発見・トラブルシュートしたい。

正しい解は?

- **A.** **knowledge base logging** を **CloudWatch Logs** 宛に設定、**CloudWatch Logs Insights** で失敗文書を query。
- **B.** CloudWatch Application Signals を有効化し KB の性能問題を自動検知・通知。
- **C.** CloudTrail を有効化し KB と ingestion 関連の全 API を追跡。
- **D.** Bedrock model invocation logging を実装し、文書処理と embedding の詳細 metric を取得。

<details>
<summary>📖 分析と解答を見る</summary>

### 🎯 この問題が試す concept
3 種の Bedrock ログを区別; 文書単位の取込追跡には **Knowledge Base logging**。Domain 5。

### 🔍 核心分析 — 「入庫検品」問題
KB が S3 から取込む時、**「Ingestion Job」** を実行: ファイルを開く → 文字を読む → chunk 化 → embed → index。この過程で **ファイル単位** のエラーが起きる（PDF 破損、パスワード保護、文字無し画像）。詳細レポートが要る: *File A - Success、File B - Failed（理由）、File C - Ignored*。

**特効薬 — Knowledge Base logging:** AWS には **KB 専用** のログ機能があり、各文書の生死（`SUCCESS`、`RESOURCE_IGNORED`、`EMBEDDING_FAILED`、`INDEXING_FAILED`）を記録し CloudWatch Logs へ送る。数万行のログは **CloudWatch Logs Insights**（ログ用 SQL）で即フィルタ: *「`INDEXING_FAILED` の全ファイルをくれ」*。

### 🧩 選択肢の分解
- **A — 正解。** Knowledge base logging はこの目的に **ぴったり**: CloudWatch Logs へ送り **ファイル単位** の状態を追跡; Logs Insights と組み合わせ高速に根本原因を troubleshoot。
- **B — 不正解（機能違い）。** CloudWatch Application Signals は APM — microservices やどの API がボトルネックか（EKS/API Gateway）を追跡。KB の取込ログは **読まない**。
- **C — 不正解（おなじみの罠）。** CloudTrail は **API 操作**（audit）のみ: 「A さんが 10:00 に `StartIngestionJob`」。だが bucket に何ファイルあり `report.pdf` が成功/失敗かは **分からない** — **document-level** の深さが無い。
- **D — 不正解（ログ種別違い）。** Model invocation logging は **chat（inference）** 時の prompt & response のみ。KB の背後の **ingestion** フェーズとは無関係。

### ✅ 結論: **A。**

### 💡 試験のコツ（3 種の Bedrock ログ — 騙されない）
1. **誰が API をいつ呼んだか** → **AWS CloudTrail**。
2. **顧客が何をチャットし AI がどう答えたか（input/output payload）** → **Bedrock Model Invocation Logging**。
3. **RAG 取込時にどのファイルが失敗したか（ingestion status）** → **Knowledge Base Logging + CloudWatch Logs Insights**。

</details>

---

## 問 14 — Amazon Q Developer で生産性最大化（2 つ選択）

> **「AI コーディング相棒」** を最大活用するには — コードを速く書き、テストも自動化しつつ、手動チェックで **流れを遅くしない** には?

ある cross-functional チームが GenAI アプリを構築、**開発者生産性の最適化**、一貫した統合パターンの強制、**性能チューニングの自動化**、**AI テストの加速** を望む。Amazon Q Developer を使用。

正しい 2 つは?（2 つ選択）

- **A.** Q Developer でセキュリティ分析するが、**全コード変更をセキュリティチームが手動承認** してから統合。
- **B.** Q Developer で、コードを書き終えた後に **遡及的（retrospective）** にパターンを分析・文書化。
- **C.** Q Developer を **統合コードの自動生成/refactor、AI コンポーネントの性能最適化提案** に設定、modular codebase 全体に適用。
- **D.** Q Developer で merge request 中の問題を修正、ただし refactor/最適化の大半は **定期的な手動レビュー周期** に回す。
- **E.** Q Developer の **自動 unit/integration test 生成** を **CI/CD pipeline** に統合。

<details>
<summary>📖 分析と解答を見る</summary>

### 🎯 この問題が試す concept
Q Developer を **proactive（能動的）+ automated（自動）** に使う、受動的/手動ボトルネックにしない。Domain 2 & 4。

### 🔍 核心分析 — 「AI アシスタントの最大活用」
Amazon Q Developer は雑学 chatbot でなく、IDE と DevOps（CI/CD）に住む **AI coding companion**。要件は難題 3 つ: (1) 生産性最適化（速く・綺麗に）; (2) 性能チューニング自動化; (3) テスト加速。達成には Q Developer を **能動的 + 自動** に使い、**手動ボトルネックを作らない**。

### 🧩 選択肢の分解
- **C — 正解（生産性 & 性能）。** コードを書きながら能動的・包括的に使う: 自動生成、綺麗に refactor、特に **性能最適化を提案** — 「性能チューニング自動化」に一致。
- **E — 正解（テスト）。** Q Developer の **自動テスト生成** を CI/CD（CodePipeline/GitHub Actions）に直結: 新コード push ごとに AI がテストを書いて実行 → 完全自動、「AI テスト加速」に一致。
- **A — 不正解（人のボトルネック）。** セキュリティ scan は良いが「**全変更を手動承認**」は巨大なボトルネックで「accelerate workflows」を破壊。
- **B — 不正解（遅く受動的）。** 「Retrospective」= コード完成後に Q Developer で読み返し + 文書化 — 受動的な「火消し」で、最初から速く書くのに役立たない。
- **D — 不正解（AI の価値を捨てる）。** AI に些末なバグだけ拾わせ、refactor/最適化は人が手動定期で — AI で労働を解放する目的に逆行。C のような AI 最適化があるのに使わず手で耕すのか?

### ✅ 結論: **C + E。**

### 💡 試験のコツ
1. **得点キーワード:** Q Developer +「productivity / accelerate」目標 → **Automated、Generate、Refactor、CI/CD での Test generation** を選ぶ。
2. **致命的キーワード:** **Manually approved**、**Manual review cycles**、**Retrospectively** を含む選択肢は即消し。AI は **自動・リアルタイム** で動かすもの。

</details>

---

## 問 15 — 画像生成に適した SageMaker inference 種別

> 適切な **「配送車」** を選ぶ: SageMaker には 4 つの inference 種別があり各々特性が違う。重い画像生成モデルは「過負荷」「timeout」を避けるためどれを使う?

ある開発者が text-to-image モデルをデプロイ（JumpStart で検証済み）。ユーザーが **オンデマンド** で画像生成するため提供したい。3 条件: **GPU 使用**、**payload 最大 40 MB**、**応答 10 分以内**。

正しい戦略は?

- **A.** **SageMaker Asynchronous Inference** を accelerated computing（GPU）instance で; Lambda がオンデマンド起動。
- **B.** SageMaker Serverless Inference を汎用 instance で; Lambda が起動。
- **C.** SageMaker Real-Time Inference を GPU instance で; Lambda が起動。
- **D.** SageMaker batch transform job を GPU instance で; Lambda が job 起動。

<details>
<summary>📖 分析と解答を見る</summary>

### 🎯 この問題が試す concept
**大 payload + 長処理 + GPU 必要 + オンデマンド** から SageMaker inference 種別を選ぶ。Domain 2 & 4。

### 🔍 核心分析 — 3 つの厳しい要件を照合
- **GPU 必要** → **Accelerated Computing** instance 必須。
- **40 MB payload** → 非常に重い（通常 API は数 MB）。
- **10 分待ち** → 非常に長い（通常 API は約 60 秒で timeout）。
- **オンデマンド** → クリックで実行、1 日末にまとめて実行ではない。

**特効薬 — Asynchronous Inference:** 重いモデル（高解像度画像生成）向け。**巨大 payload（最大約 1 GB）** — 40 MB は余裕 — と **長い処理時間** — 10 分は余裕 — を許容。オンデマンド要求を捌く **内部キュー** もある。

### 🧩 選択肢の分解
- **A — 正解。** Asynchronous Inference は GPU instance（accelerated computing）、大 payload、長処理に対応、オンデマンド用の internal queue あり → 40 MB と 10 分の両方に一致。
- **B — 不正解（GPU 無し）。** Serverless Inference はサーバ管理不要で便利だが **致命的弱点は GPU 非対応**。GPU 無しの画像生成は不可能か激遅。
- **C — 不正解（過負荷 & timeout）。** Real-Time は **超高速（ミリ秒）** 向けで厳しく制限: 最大 payload **約 6 MB**（40 MB 必要）、timeout **約 60 秒**（10 分必要）→ 即エラー。
- **D — 不正解（offline で不適）。** Batch transform は **offline 一括処理** — 末にファイルを集め一括実行。クリックで即画像が欲しいインタラクティブ用途（オンデマンド）向けでない。

### ✅ 結論: **A。**

### 💡 試験のコツ（SageMaker inference チートシート）
- **Real-time:** ミリ秒応答、小 payload（約 6 MB）、短 timeout（約 60s）→ ライブ chatbot。
- **Asynchronous:** 巨大 payload（約 1 GB）、長処理、内蔵 queue → **画像生成（computer vision）、動画処理**。
- **Serverless:** サーバ管理不要、spiky traffic、ただし **GPU 非対応**。
- **Batch Transform:** offline 処理、夜間に大きなデータ塊を分析、インタラクティブ用途は不可。

</details>

---

## 問 16 — 大規模・低頻度の vector search、最安

> **「3 年養って 1 時間使う」** タイプのコスト最適化: 膨大なデータを保存し **たまにしか検索しない**、サーバ維持費で「火の車」にならずに。

ある地球観測アプリが **8,000 万枚の衛星画像** で similarity search を行い、毎日新規取込、だが検索は **低頻度（infrequently）** — アナリストが比較する時だけ。要件は **最安**、応答が速く、**インフラ管理なし**。

正しい解は?

- **A.** vector を OpenSearch Serverless に保存、vector search を使う。
- **B.** vector を DynamoDB に保存、similarity ロジックを Lambda で自作。
- **C.** **Amazon S3 vector bucket** を vector index 付きで作成し、embedding を保存・類似検索。
- **D.** vector を RDS for PostgreSQL + pgvector に保存。

<details>
<summary>📖 分析と解答を見る</summary>

### 🎯 この問題が試す concept
大量だが **検索が疎** な場合に **最安** の vector ストア — 常時稼働で維持費を取る store ではない。Domain 1 & 4。

### 🔍 核心分析 — キラーキーワード「infrequently」
- **常時稼働の DB**（RDS）や **背後で計算能力を維持** する仕組み（OpenSearch Serverless）を借りると、1 週間誰も検索しなくても月額を払う → 大きな無駄。
- 第 2 要件: **インフラ管理なし** — RAM/CPU 選定も server 設定も不要。

**特効薬 — Amazon S3 Vectors:** S3 の **真の serverless** 機能。数百万の vector を放り込み、誰も検索しない時は **激安** の S3 ストレージ費だけ。アナリストが「検索」を押した時だけ AWS がそのクリックに課金（**pay-as-you-go**）→ **低頻度 + 最安** に最適。

### 🧩 選択肢の分解
- **C — 正解。** S3 Vectors は managed（インフラ管理なし）、**数十億 vector** まで拡張。S3 の **pay-as-you-go** ゆえ **infrequent searches** に特にコスト効率的 — アイドル時にサーバを「養う」費用が不要。
- **A — 不正解（base capacity 課金）。** OpenSearch Serverless の k-NN vector search は強力だが **継続的・高頻度** 検索向け設計。"serverless" でも準備状態維持に **最低 OCU（OpenSearch Compute Units）** 課金 → 検索が稀だと背後 OCU が無駄。
- **B — 不正解（道具違い）。** DynamoDB は優れた Key-Value だが **vector search 非搭載**。Lambda で 8,000 万件を scan し距離を自前計算 → 激遅 + 巨額の Lambda 計算費。
- **D — 不正解（サーバ管理 + 維持費）。** RDS + pgvector は DB server（db.m5.large…）の **provision** が必要、アイドルでも **時間課金** + 保守 → 両要件に違反。

### ✅ 結論: **C。**

### 💡 試験のコツ
1. **「Infrequently」+「Cost-effective」+「Vector Search」** → vector 対応の最安ストレージ: **Amazon S3 Vectors**。
2. **常時稼働 DB を除外:** 「たまにしかユーザーが居ない」なら RDS、Aurora、OpenSearch（Serverless でも）を消す。これらは **継続的/高頻度** トラフィックでこそ cost-effective。

</details>

---

## 問 17 — どの guardrail ルールがブロックしたか特定

> **「AI 捜査官」**: システムが顧客を拒否した時、**なぜ** ブロックしたか（暴言、禁止トピック、ID 漏洩）を正確に知るには?

Bedrock 上の AI アシスタントに複数の guardrail（prompt injection、機密情報フィルタ、denied topic ブロック）。クエリがブロックされた時、開発者は **どの guardrail ルールが発動したか詳細分析** して fine-tune し、正当なクエリと真の脅威を区別したい。

最も詳細な分析を与える設定は?

- **A.** Bedrock model evaluation を自動評価 job + guardrail assessment metrics で有効化。
- **B.** model invocation logging を有効化、`InvocationsIntervened` を `GuardrailContentSource` でフィルタし CloudWatch alarm。
- **C.** **guardrail tracing**（`{"trace": "enabled"}`）を設定、`InvocationsIntervened` を **`GuardrailContentSource`** でフィルタ監視。
- **D.** **guardrail tracing**（`{"trace": "enabled"}`）を設定、`InvocationsIntervened` を **`GuardrailPolicyType`**（ContentPolicy、TopicPolicy、SensitiveInformationPolicy）でフィルタ監視。

<details>
<summary>📖 分析と解答を見る</summary>

### 🎯 この問題が試す concept
Guardrails のトラブルシュート: **trace** を有効化し正しい **dimension** でフィルタして **どのルール** が介入したか知る。Domain 3。

### 🔍 核心分析 — 「犯人探し」問題
Bedrock Guardrails は **多層の関所**（複数の「警備員」）:
1. **「有害語」** を見る警備員（Content Policy）。
2. **「禁止トピック — 例: 政治」** を見る警備員（Topic Policy）。
3. **「PII 漏洩」** を見る警備員（Sensitive Information Policy）。

顧客が「弾かれた」時、**どの警備員が動いたか** を正確に知る必要。そのために: (1) **「trace」** を有効化して審査ログを記録; (2) ログを **Policy Type** でフィルタして介入した層の正確な名前を知る。

### 🧩 選択肢の分解
- **D — 正解。** `{"trace": "enabled"}` は詳細レポートを出させる必須設定。最も重要なのは **`GuardrailPolicyType`** でフィルタすると **どのルールが違反したか**（ContentPolicy / TopicPolicy / SensitiveInformationPolicy）が正確に分かる → 開発者が fine-tune の道筋を得る。
- **C — 不正解（焦点違い）。** tracing は有効だが **`GuardrailContentSource`** でフィルタ — これは **input（質問）** か **output（回答）** のどちらでブロックされたかだけで、**どのルールか** は分からない。
- **B — 不正解（tracing 欠如）。** model invocation logging は input/output を記録するが Guardrails の内部判断の **詳細 JSON 構造** を提供しない; さらに C と同じ誤った dimension。
- **A — 不正解（道具・時期違い）。** Model Evaluation は **testing/pre-production** でモデルの賢さを採点するもの、**リアルタイム** 本番クエリの監視/トラブルシュート用ではない。

### ✅ 結論: **D。**

### 💡 試験のコツ
1. **どのルール** でブロックしたか → `GuardrailPolicyType`（Topic / Content / PII）。
2. **質問か回答か** でブロックしたか → `GuardrailContentSource`（Input / Output）。
3. 詳細分析には常に Converse API 設定に **`{"trace": "enabled"}`** が必要。

</details>

---

## 問 18 — 安全な認証、IdP federation、long-lived credentials なし（2 つ選択）

> 重要なエンタープライズ知識: スタッフ/アプリが AI を安全に使えるように **パスワードやアクセスキー漏洩の心配なく** — **「使い捨ての入場パス」** で。

ある企業が Bedrock を使うサードパーティアプリに安全な認証が必要。解は: **既存 IdP（Identity Provider）と統合**、**包括的な audit log** を保持、**long-lived credentials を排除**、Bedrock への **一時アクセス** を提供。

正しい 2 つは?（2 つ選択）

- **A.** **OIDC を Amazon Cognito** と統合: IdP 経由で認証し token を一時 AWS credentials に交換して Bedrock へアクセス。
- **B.** API Gateway の Lambda authorizer が LDAP で検証し JWT を発行して Bedrock アクセス。
- **C.** 従業員ごとに IAM user を作成、IAM policy で権限付与、Secrets Manager で rotation。
- **D.** IAM role + STS AssumeRole で federation、**アプリの IAM user credentials を設定ファイルに保存**。
- **E.** **AWS IAM Identity Center を SAML federation** で IdP に展開、permission set で Bedrock アクセスを付与。

<details>
<summary>📖 分析と解答を見る</summary>

### 🎯 この問題が試す concept
IdP から **一時 credentials** を発行する **Identity Federation**、静的キーを排除。Domain 3。

### 🔍 核心分析 — 「使い捨て入場パス」問題
クラウドセキュリティで最も危険な敵は **long-lived credentials**（永続的な Access Key & Secret Key）。永続キーを作りソースコードに埋め、ハッカーがソースを盗めば → システムは **永遠に** 侵害。

**特効薬 — Identity Federation:** 標準は **AWS 内部アカウントを作らず**、既存の人事/ID システム（Okta、Microsoft Entra ID などの IdP）と AWS を繋ぐ。従業員が IdP に成功ログインすると、AWS が数時間だけ有効な **一時 credentials** を発行 — 時間切れで自動破棄。静的キーは保存されず、盗まれても無価値。

### 🧩 選択肢の分解
- **A — 正解（アプリ向け）。** Amazon Cognito は **アプリ（web/app）** のサインインを管理、**OIDC** で会社 IdP と federation。最大の利点: Cognito Identity Pools が **IdP token を一時 AWS credentials に交換** して Bedrock を呼ぶ; 全操作は CloudTrail が自動 log。
- **E — 正解（社内 workforce 向け）。** **AWS IAM Identity Center**（旧 AWS SSO）は会社 ID システムと **SAML 2.0 federation** する究極ツール: 完全に **一時 credentials** で権限付与（long-lived credentials を排除）、全サインイン/API 呼び出しが CloudTrail に log。
- **B — 不正解（車輪の再発明、安全性低）。** 旧式 **LDAP** に繋ぐ Lambda を自作し **JWT** を発行する custom は危険; Bedrock API は **custom JWT を直接受けず**、AWS credentials への交換が要る。
- **C — 不正解（核心要件違反）。** **IAM Users** を人ごとに作る = **long-lived credentials**（永続パスワード & Access Key）を作る — 悪名高い anti-pattern、「long-lived credentials 排除」に直接違反。Secrets Manager の rotation も静的キーの本質を変えない。
- **D — 不正解（重大なセキュリティ欠陥）。** 前半の STS AssumeRole（一時 credentials 発行）は正しいが、後半が **「IAM user credentials を設定ファイルに保存」** を推奨 — 静的キーをアプリに hardcode は **致命的欠陥**、要件に完全違反。

### ✅ 結論: **A + E。**

### 💡 試験のコツ
1. **「Eliminate long-lived credentials」** → **IAM Users** 作成や **Access Keys** 保存の選択肢は即消し。
2. **一時 credentials を発行する 2 つの federation サービス:** **AWS IAM Identity Center（SAML）** は **社内 workforce** の AWS アクセス向け; **Amazon Cognito（OIDC/SAML）** は **customer-facing アプリ** 向け。このセキュリティパターンでは両方が正解。

</details>

---

## 問 19 — ピーク時 throttling、同一 FM、最安

> 成功しすぎたシステムの「病気」: **ピーク時の渋滞（throttling）**。大金を払わず、負荷分散コードも自前で書かずに、AI の提供上限を広げるには?

ある EC プラットフォームが Bedrock で商品説明を生成、アプリは **1 つの Region** にある。ピーク時に「Too many requests, please wait before trying again」。ピーク時に **throughput を増やす** が **運用 overhead を増やさない**、**既存 Bedrock API と互換維持**、**同一 FM を使用**。要件は **最安**。

正しい解は?

- **A.** Lambda が既定で元 Region の Bedrock を呼び、エラー時に副 Region へ fallback。
- **B.** **Cross-Region Inference** で地理エリア内の複数 Region にトラフィックを分散。
- **C.** prompt routing で同一ファミリーの複数 FM にトラフィックを分散。
- **D.** Provisioned Throughput で FM に高い throughput を割り当て。

<details>
<summary>📖 分析と解答を見る</summary>

### 🎯 この問題が試す concept
ピーク時の quota 上限を **Cross-Region Inference** で超える、on-demand、コード変更なし。Domain 1 & 4。

### 🔍 核心分析 — 「ピーク時の渋滞」問題
各 Region（例 `us-east-1`）には毎分の API 呼び出し **quota** がある。Flash Sale でリクエストが急増し、その Region の Bedrock が詰まる → rate limit / throttling エラー。

**特効薬 — Cross-Region Inference:** 「Region A がエラーなら Region B」の複雑なコードを開発者に書かせる代わり、Bedrock のこの機能: **Model ID** を **Inference Profile ARN** に変えるだけで、AWS が交通整理役に — *「Region A が渋滞、空いている Region B に自動で振り、結果を返す、料金は同じ!」*。**運用コードゼロ** で quota が倍増。

### 🧩 選択肢の分解
- **B — 正解。** Cross-Region Inference は全要件を満たす: 複数 Region の quota を束ねて throttling 回避; **同一 FM** のまま; 複雑なコード/インフラ **不要**; **実使用 token（on-demand）** 課金 → スパイクに **最安**。
- **D — 不正解（「金持ち」の解、超高価）。** Provisioned Throughput は GPU を月間 **丸抱え**。throttling は解決するが **超高額**（月数万 USD）。**ピーク時** だけ詰まる会社に 24/7 サーバ買い切りは大浪費、"cost-effective" に逆行。
- **A — 不正解（車輪の再発明）。** Lambda の Try/Catch で他 Region に failover は Cross-Region Inference **登場前** のやり方; コード保守・エラー追跡で **運用 overhead** が発生。
- **C — 不正解（技術要件違反）。** Intelligent Prompt Routing は質問の **難易度** で振る（易→Haiku で安く、難→Sonnet）。難点は **複数の異なる FM** を使う点 → **「同一 FM 使用」** に露骨に違反。

### ✅ 結論: **B。**

### 💡 試験のコツ
1. **「Too many requests / throttling limits / peak spikes」+「cost-effective」** → 解は常に **Cross-Region Inference**（quota を弾力的に拡張、従量課金）。
2. **Provisioned Throughput:** **高く・安定し・継続的 24/7** の負荷（consistent, predictable workload）で長期コミットする時だけ。「spikes」「peak periods」があれば絶対選ばない。

</details>

---

## 問 20 — 検索に入れる前に PII を redact

> 金融/銀行の機微な問題: AI に渡す前に **PII（Personally Identifiable Information — 個人識別情報）** を処理。数百万の古いメールから電話番号・口座番号・顧客名を、AI が読む前に **「洗浄」** するには?

ある銀行が口座情報照会を助けるモバイルアプリを作りたい、顧客とサポート担当の **メール**（S3 保存）をソースに使用。データに **検索結果に出てはいけない PII** が含まれる。

正しい解は?

- **A.** Kendra で S3 のメールを検索、Bedrock FM を統合、**system prompt** でクエリ処理時に PII を除去。
- **B.** **Amazon Comprehend** で S3 のメールから PII を検出 + **redact（伏せ字化）**、**Amazon Kendra** と統合して処理済みデータを検索。
- **C.** Textract でテキスト抽出、Macie で S3 の PII を scan、Kendra と統合して検索。
- **D.** Comprehend で PII を redact、**DocumentDB** と統合して検索。

<details>
<summary>📖 分析と解答を見る</summary>

### 🎯 この問題が試す concept
**先に洗浄、後で検索** のアーキ: PII を redact（Comprehend）→ 検索用に index（Kendra）。Domain 3。

### 🔍 核心分析 — 「データ洗濯」問題
メール記録には機微データ（氏名、カード番号、住所）が満載。**セキュリティ第 1 原則**（特に金融）: **未洗浄の raw data を検索/AI システムに絶対入れない**。すべて **redact（伏せ字化）** する「洗濯機」が必要（例「口座 123456」→「口座 [ACCOUNT_NUMBER]」）。

**特効薬 — Comprehend + Kendra:**
1. **Amazon Comprehend（洗濯機）:** **PII Redaction** を持つ NLP サービス — 数百万メールを読み、カード番号/氏名を自動検出して `***` や匿名ラベルに置換。
2. **Amazon Kendra（賢いファイルキャビネット）:** Comprehend が「洗浄」後、清浄データを Kendra へ — 強力な **semantic enterprise search** エンジン。ユーザーは他人の PII 漏洩リスクゼロで自由に検索。

### 🧩 選択肢の分解
- **B — 正解。** Comprehend（PII 検出 + redact）+ Kendra（index + enterprise search）が完璧・安全・完全自動の解。
- **A — 不正解（致命的なセキュリティミス）。** **system prompt** に PII 保護を任せるのは超リスク: LLM は確定的フィルタでなく、**jailbreak** や幻覚で情報を漏らし得る。機微データは AI が見る **前に redact** すべき。
- **C — 不正解（道具が全て違う）。** **Textract** は画像/スキャン PDF から文字を読む OCR — メールは元々テキストで無意味。**Macie** は **監査** ツール: 「この bucket に PII あり」と **警告** するだけ、AI 用に **redact しない**。
- **D — 不正解（検索場所違い）。** 前半の Comprehend は正しいが、後半が **DocumentDB**（MongoDB 互換の JSON DB）を検索ツールに。DocumentDB は Kendra のような **自然言語理解** を持たず chat 検索できない → アーキ違い。

### ✅ 結論: **B。**

### 💡 試験のコツ（道具チートシート）
- **Amazon Comprehend** — NLP: テキストを読み **PII を redact** できる **唯一** の道具。
- **Amazon Kendra** — semantic enterprise search: 社内文書を自然言語で検索。
- **Amazon Macie** — セキュリティ発見/評価: PII を **警告** するだけ、**redact しない**。
- **Amazon Textract** — 画像/PDF の OCR: メールのような raw text には不要。
- **Amazon DocumentDB** — NoSQL JSON: 自然言語検索エンジンではない。

</details>

---

> **20 問終わり。** これは概念練習用に著者が書いたオリジナルコンテンツで、実試験問題ではありません。[DISCLAIMER](../../DISCLAIMER.md) を参照。
