# Practice Questions (Original)

[← Back to Practice Exam](./README.md)

> 20 **original questions written by the author**, rewritten from the *concept* being tested (different industry, context, and numbers from any real exam). **Not real exam questions, not copied from AWS Skill Builder.** See [DISCLAIMER](../../DISCLAIMER.md).
>
> **How to use:** read the scenario → pick your answer first → open "View analysis & answer". The **Core analysis**, **Option breakdown**, and **Exam tip** matter more than getting it right or wrong.

---

## Question 1 — RAG: the best result sinks to the bottom (Select TWO)

> This touches a very common **"disease"** in RAG (Retrieval-Augmented Generation) systems: the system retrieves a whole basket of relevant documents, but the **best one sinks to the very bottom**. How do you push it to the top with the **least effort**?

A telecom company builds a RAG application on Amazon Bedrock to look up network-troubleshooting documents. The system retrieves many relevant passages, but technicians report that the most useful passage often appears at the bottom of the list. The company wants to improve result ranking with **minimal operational overhead**.

Which two steps meet the requirement? (Select TWO.)

- **A.** Enable **hybrid search** on Knowledge Bases (combine vector embeddings with keyword matching), using OpenSearch Serverless as the backend.
- **B.** Self-install the Learning to Rank plugin on OpenSearch, collect user click data, and train a scoring model.
- **C.** Use a **managed Amazon Bedrock reranker model** to reorder results by semantic relevance.
- **D.** Create Aurora PostgreSQL + pgvector and write a custom similarity-scoring algorithm.
- **E.** Use SageMaker JumpStart + Kendra Intelligent Ranking to build a custom scoring algorithm.

<details>
<summary>📖 View analysis & answer</summary>

### 🎯 What concept this tests
Improving RAG **retrieval** quality with **built-in managed features** instead of building your own. Domain 1 (FM Integration & Data).

### 🔍 Core analysis — the "panning for gold" problem
When RAG searches the corpus, it pulls out a basket of documents (say 20 passages). The trouble is the passage with the best answer sits at position 19. Reading top-down, the model gets **"information overload"** and misses it — the classic *"Lost in the middle"* phenomenon.

**The cure — the "Hybrid Search + Reranker" combo:** AWS provides two ready-made weapons to push the best passage to the top with almost no code:
1. **Hybrid Search:** instead of pure vector search (fuzzy semantics), combine **keyword** matching (exact words). The two methods complement each other and automatically surface the most precise documents. On Bedrock Knowledge Bases this is just a **configuration toggle** — effortless.
2. **Reranker Model:** acts as a "judge". After retrieving 20 passages, hand all 20 to a reranker (e.g. Cohere Rerank on Bedrock). The judge re-scores and reorders them, lifting the best passage to #1. It's **built-in** — no training required.

### 🧩 Option-by-option breakdown
- **A — CORRECT.** Moving from pure vector to hybrid search is the fastest, most effective way to boost RAG accuracy. Configuring it on Knowledge Bases (OpenSearch Serverless backend) is purely **enabling a built-in feature** — no algorithm to write, very low effort.
- **C — CORRECT.** The Bedrock reranker is built **precisely** for "good results buried below"; it reorders automatically. Being managed, it fully meets *minimal operational overhead* — just call the API.
- **B — WRONG (too much work).** Learning to Rank is an open-source plugin: you must **collect click data yourself**, **train** a scoring model yourself, and **maintain** it. Huge overhead, violates the requirement.
- **D — WRONG (reinventing the wheel).** pgvector is fine, but forcing developers to **"write a custom similarity-scoring algorithm"** is painful: hand-coded math, infra to manage — the opposite of "minimal overhead".
- **E — WRONG (overengineered).** Stitching SageMaker JumpStart + Kendra and then "building a custom algorithm" is far too heavy when Bedrock already has a reranker built in.

### ✅ Conclusion: **A + C.**

### 💡 Exam tip
1. **"Improve ranking / most relevant lower / better retrieval"** → the undefeated AWS pairing is **Hybrid Search** (vector + keyword) and **Reranker Models** (re-score + reorder).
2. **Deadly keyword "MINIMAL overhead":** immediately cross out any option with *Create custom algorithm*, *Train custom model*, or *Learning to Rank plugin*. Always prefer Bedrock's **built-in/managed** features.

</details>

---

## Question 2 — Real-time and resilient Knowledge Base sync

> A classic architecture pattern: **handling real-time events resiliently** (event-driven & resilient). How do you have the AI update documents instantly **without crashing** when many files change at once?

A hospital runs an AI assistant that answers from internal procedure documents, using Amazon Bedrock Knowledge Bases with Amazon S3 as the source. New documents must appear in answers **as soon as possible**, and deleted documents must be excluded **as soon as possible**. The solution must be **scalable, event-driven, and resilient**.

Which solution meets the requirements?

- **A.** EventBridge Scheduler runs a Lambda every 5 minutes to detect S3 changes and call the add/delete document APIs.
- **B.** S3 Event Notifications invoke a Lambda directly on object-created/deleted; the Lambda calls `IngestKnowledgeBaseDocuments` / `DeleteKnowledgeBaseDocuments`.
- **C.** S3 Event Notifications send created/deleted events to an **Amazon SQS** queue; a Lambda polls the queue and calls `IngestKnowledgeBaseDocuments` / `DeleteKnowledgeBaseDocuments`.
- **D.** EventBridge Scheduler runs a Lambda every 5 minutes that calls `StartIngestionJob` to resync everything.

<details>
<summary>📖 View analysis & answer</summary>

### 🎯 What concept this tests
An **event-driven + buffering with SQS** architecture that is both real-time and resilient to load spikes. Domain 2 (Implementation & Integration).

### 🔍 Core analysis — "update super-fast but never crash"
The prompt plants three keywords, each ruling out options:
1. **"As soon as possible":** must be real-time — update the moment a file lands → **rule out any Scheduled/Cron approach**.
2. **"Event-driven":** a file change (event) auto-triggers the system.
3. **"Resilient" — the money keyword:** if the hospital uploads 1,000 files in one second, Bedrock gets overwhelmed and **throttles requests**. To be resilient you **must** put a **"funnel/buffer"** in the middle to hold those 1,000 files and feed them to Bedrock gradually.

**The cure — Amazon SQS:** the SQS queue is that funnel. It catches every S3 event, keeps it safe without losing data, and has a **built-in retry** if the Lambda's Bedrock update fails. The Lambda pulls messages out one at a time and processes at a steady rate, no overload.

### 🧩 Option-by-option breakdown
- **C — CORRECT.** The chain `S3 → SQS → Lambda → Bedrock API` is an AWS best practice: S3 Event Notifications provide "event-driven" and "real-time"; SQS provides "resilient" (a shock-absorbing buffer + auto-retry); the system scales freely without crashing.
- **A — WRONG.** Running every 5 minutes violates "as soon as possible" and isn't event-driven. Making the Lambda **"detect changes"** itself (diff old/new files) is resource-heavy and inefficient.
- **B — WRONG (dies for lack of resilience).** `S3 → Lambda` directly is fast and event-driven, **but has no buffer**. On a spike, Bedrock throttles → Lambda fails → with no queue to hold the failed event, that file is **never updated** (data loss).
- **D — WRONG.** Slow (5 min) and uses the wrong API: `StartIngestionJob` **rescans and resyncs the entire** S3 store — much slower and pricier than incrementally adding/deleting just the changed file (`IngestKnowledgeBaseDocuments`).

### ✅ Conclusion: **C.**

### 💡 Exam tip
1. **"As soon as possible / near real-time" + "Resilient" / "Traffic spikes":** the correct answer almost always includes **Amazon SQS** as a buffer (decoupling/buffering).
2. **Direct invocation (S3 → Lambda):** drop it immediately if the prompt stresses "resilient" or "traffic spikes".
3. **Scheduler / Cron job:** cross out at once if the prompt says "as soon as possible" or "event-driven". Scheduling is the **enemy of real-time**.

</details>

---

## Question 3 — Analyze images/video with the least effort

> Giving the AI **"eyes"**: analyze photos and videos, then summarize trends on a dashboard — without the IT team having to **slave away** labeling data and training models.

A sports-media company wants to analyze **videos and photos** from public matches to understand tactical elements and trends, store the extracted information, and build a summary dashboard. Requirement: **least operational overhead**.

Which solution meets the requirements?

- **A.** EventBridge triggers Lambda that uses QuickSight Q to analyze videos/photos, stores in S3, builds a QuickSight dashboard.
- **B.** Use **Step Functions** to orchestrate video/photo processing with **multimodal FMs on Amazon Bedrock**, store in S3, visualize with **QuickSight**.
- **C.** Use Rekognition Custom Labels to train a custom model, store in DynamoDB, dashboard via Managed Grafana + custom plugins.
- **D.** Use Claude to analyze text descriptions, use Stable Diffusion to "analyze" images, store in OpenSearch, dashboard via Grafana.

<details>
<summary>📖 View analysis & answer</summary>

### 🎯 What concept this tests
Letting AI understand image/video content with a **built-in multimodal Foundation Model** (no custom training), plus a managed dashboard. Domain 1 & 2.

### 🔍 Core analysis — "AI with eyes"
The input is **video and photos** (visual data). For the AI to "see" them you need a brain with **multimodal** capability. The prompt also has the killer keyword: **LEAST operational overhead** — meaning pick ready-made services, **no self-training**, no server management, no custom UI code.

**The cure — Bedrock Multimodal + QuickSight combo:**
1. **Multimodal FMs** (Claude, Amazon Nova Pro): read text and "see" images/video. No extra training — they already understand the tactical pattern a player is running.
2. **Amazon QuickSight:** a serverless drag-and-drop BI/dashboard tool — effortless for a trends report.

### 🧩 Option-by-option breakdown
- **B — CORRECT.** Bedrock multimodal FMs (ready-made) let the AI "watch" video directly with no labeling/training; Step Functions orchestrating the extraction is best practice; QuickSight builds the dashboard straight from S3 data (via Athena) with no complex software or plugins.
- **A — WRONG (misreads QuickSight Q).** QuickSight **Q** is a natural-language Q&A feature over **tabular/numeric data** (e.g. "what was this month's revenue?"). It **cannot "see"** `.mp4`/`.jpg` files to analyze imagery.
- **C — WRONG (high operational overhead).** Rekognition Custom Labels = your team uploads thousands of images, **draws bounding boxes**, labels, and **trains**; then **writes custom plugins** for Grafana. Enormous effort, violates "least overhead".
- **D — WRONG (seriously wrong tool).** **Stable Diffusion is an image-generation model (text-to-image)** — used to *create* new images, **not** to *analyze/recognize* existing ones. Self-managing an OpenSearch cluster also adds operational work.

### ✅ Conclusion: **B.**

### 💡 Exam tip
1. **"Analyze videos and photos / image understanding / no custom training"** + LEAST overhead → pick **Multimodal FMs on Bedrock** (Claude, Nova).
2. **Rekognition Custom Labels:** only when the prompt explicitly says "the company has its own labeled image set + wants to recognize proprietary logos/products". Generic object recognition → use a ready-made service.
3. **"Dashboard / Business Intelligence" the easy way (serverless/managed)** → **Amazon QuickSight** is the default answer.

</details>

---

## Question 4 — Order the model-replacement evaluation workflow

> An **ordering** question with a very real scenario: how do you **"change generals mid-battle"** — replace an old AI model serving customers with a new one — without breaking the system?

A company wants to replace its production translation model with a new one, **only if the new model proves better**. The process must be **sequential**, with each step reviewed and approved before the next. **Order** the following steps:

- Analyze the results and generate a comprehensive evaluation report.
- Conduct A/B testing comparing the new model against the current production model.
- Create a test dataset with diverse scenarios and edge cases.
- Define evaluation metrics (relevance, factual accuracy, fluency).
- Implement automated quality gates using AWS Step Functions.

<details>
<summary>📖 View analysis & answer</summary>

### 🎯 What concept this tests
A systematic evaluation process — **scientific and sequential**, set the yardstick first, then measure. Domain 5 (Testing, Validation & Troubleshooting).

### 🔍 Core analysis — "running a talent exam"
Swapping in a new AI is very risky; you can't just "feel" it's smarter and ship it. AWS makes you follow a principle like organizing an exam: **set grading rules → set the test paper → hold the real exam → grade through automated gates → make the final decision**.

### ✅ Correct order (and why)
1. **Define evaluation metrics** — before anything, know **"what good looks like"** (target accuracy %, fluency). This is the yardstick for every later step.
2. **Create test dataset** — with grading rules set, prepare the **"test paper"**: a dataset with hard cases and edge cases, shared to test both models.
3. **Conduct A/B testing** — **"into the ring"**: both the old model (A) and new (B) take the test from step 2, measured against the metrics from step 1.
4. **Implement automated quality gates (Step Functions)** — the **"machine judge"**: Step Functions auto-check whether the new model clears the threshold, and **force human approval** before proceeding.
5. **Analyze results & generate report** — after clearing every gate, **compile the evidence** into a report so leadership can hit "approve the swap".

### 💡 Exam tip
For evaluation workflow/ordering questions, remember the mantra: **Metrics → Dataset → Test → Gate/Threshold → Report**. Never feed data into testing before the evaluation metrics are defined.

</details>

---

## Question 5 — Enforce Guardrails on every call

> An **enterprise AI-governance** problem: how do you **"force"** every developer to route AI calls through the safety filter (Guardrails) without having to **police each one** individually?

A manufacturing conglomerate rolls out an AI governance policy: **every** `InvokeModel` and `Converse` call to an FM **must** apply Bedrock Guardrails. Developers across teams might "forget" to attach a guardrail in code. The solution must be **MOST operationally efficient**.

Which solution?

- **A.** Configure IAM policy with both `bedrock:GuardrailIdentifier` **and** `bedrock:PromptRouterArn` condition keys, requiring prompt-router validation before access.
- **B.** Write a Lambda that validates and enforces guardrails before proxying requests to Bedrock; force all interactions through the Lambda.
- **C.** Store the guardrail ID in Parameter Store; a Lambda fetches the ID each time before calling Bedrock.
- **D.** Configure IAM policy with the **`bedrock:GuardrailIdentifier`** condition key, applied to every IAM role that accesses the FMs.

<details>
<summary>📖 View analysis & answer</summary>

### 🎯 What concept this tests
Using **IAM Policy + Condition Key** to enforce compliance at the **infrastructure layer** — instead of writing code to police other people's code. Domain 3.

### 🔍 Core analysis — "forcing compliance"
A developer can easily "forget" or deliberately skip inserting the guardrail ID in code. Without the ID, the AI responds with **no safety filter** (no profanity blocking, no PII protection).

**The cure — the IAM checkpoint:** the smartest, lowest-effort governance isn't writing code to inspect code, it's an **IAM Policy with a Condition Key**. You set a rule at the AWS gate: *"Anyone calling the Bedrock API whose request does NOT contain `bedrock:GuardrailIdentifier` → I (AWS) reject it immediately with Access Denied!"*. This is **free, built-in, and zero maintenance**.

### 🧩 Option-by-option breakdown
- **D — CORRECT (gold standard).** Using the `bedrock:GuardrailIdentifier` condition key in the IAM policy is the official, most efficient way. Any call without a guardrail ID is **blocked at the infrastructure level** by IAM. No custom code → highest operational efficiency.
- **A — WRONG (gilding the lily).** It has the right condition key but also forces `bedrock:PromptRouterArn`. Prompt Router is for **routing prompts**, entirely **unrelated** to enforcing guardrails; adding irrelevant conditions only complicates the policy needlessly.
- **B — WRONG (self-made bottleneck).** A Lambda proxy between users and Bedrock can **crash, overload, add latency** to every query, and needs constant code maintenance → severely violates "operationally efficient".
- **C — WRONG (slow and costly).** Like B it loses points for using Lambda; worse, **every call** has to round-trip to Parameter Store to fetch the ID → pointless latency + thousands of Parameter Store API charges.

### ✅ Conclusion: **D.**

### 💡 Exam tip
1. **"Enforce Bedrock Guardrails" + "MOST operationally efficient"** → the answer is always **IAM Policy + the `bedrock:GuardrailIdentifier` condition key**.
2. **Stay away from custom proxies:** when AWS asks for Access Control / Governance, always use **IAM**. Cross out any option pushing a **Lambda proxy** in the middle to check permissions — it's poor design against AWS's serverless-governance philosophy.

</details>

---

## Question 6 — Stop generation at a specific phrase

> About **inference parameters** — specifically the **"brake pedal" of the AI**: how do you make it **stop instantly** the moment it writes a predefined phrase?

A developer builds a virtual assistant with Anthropic Claude on Bedrock. The app needs to **stop generating output right after a specific phrase is produced** in the response.

Which solution?

- **A.** Add "stop at this phrase" to the user prompt.
- **B.** Use the **stop sequences** parameter in the inference call to specify the trigger phrase.
- **C.** Use the **top-k** parameter to control token diversity.
- **D.** Use the **temperature** parameter to control how likely the phrase is to appear.

<details>
<summary>📖 View analysis & answer</summary>

### 🎯 What concept this tests
Understanding each inference parameter; recognizing that **stop sequences** is the reliable stopping mechanism. Domain 2.

### 🔍 Core analysis — the "brake pedal"
You want the AI to stop the instant it writes "Conclusion:".
- **The naive way:** tell it in the prompt — like telling a driver to stop verbally; sometimes the driver **"overshoots"**. LLMs often "blurt past", especially at high creativity.
- **The cure — the automatic breaker (stop sequences):** the Bedrock API has `stop_sequences = ["Conclusion:"]`. This is the **server's automatic brake**: the instant the server sees the model about to emit that string, it **cuts the connection immediately**, guaranteeing 100% the AI says no more.

### 🧩 Option-by-option breakdown
- **B — CORRECT.** Stop sequences are built exactly for this. You pass an array of words/characters (e.g. `["\n\nHuman:", "Conclusion:"]`); when the model is about to generate that string, token generation **halts instantly**. A 100%-reliable technical control.
- **A — WRONG (depends on the AI's mood).** Prompt instructions rely on the model's willingness to obey. An LLM isn't a perfect if-else machine; mid-flow it may ignore the instruction and keep generating → unreliable for a core feature.
- **C — WRONG (misreads Top-k).** Top-k limits the **vocabulary size** the model considers for the next token (e.g. pick 1 of the 50 highest-probability words); it affects diversity, **has no "stop" ability**.
- **D — WRONG (misreads Temperature).** Temperature controls **randomness/creativity** (high = flowery, low = formulaic); it never halts or stops generation.

### ✅ Conclusion: **B.**

### 💡 Exam tip (4 classic parameters)
1. Make the AI **STOP** at a specific character/word → **stop sequences**.
2. Reduce **hallucination**, keep answers serious and stable → **lower temperature toward 0**.
3. Make the AI **creative**, write poems/stories → **raise temperature** toward 1.
4. Control the AI's **vocabulary** (avoid weird words) → tune **Top-P / Top-K**.

</details>

---

## Question 7 — Optimize LLM endpoint resource utilization (Select TWO)

> An **LLM deployment cost/hardware optimization** problem: like renting a **50-seat bus to carry only 5 people** — hugely wasteful! How do you fill that bus?

A fine-tuned LLM is deployed to a SageMaker AI endpoint with continuous batching (the DJL library). Each EC2 has **8 GPUs**. In production it needs too many instances → rising cost. Logs show: (1) the actual I/O sequence length is **10× smaller** than configured, and per-instance concurrency is low; (2) weights + activations fit entirely within **4 GPUs**.

Which two steps improve resource utilization? (Select TWO.)

- **A.** Increase the number of SageMaker instances and spread requests evenly.
- **B.** **Reduce the model's maximum sequence length** to allow a higher rolling batch size per GPU.
- **C.** Enable speculative decoding to reduce per-request latency.
- **D.** Use **tensor parallelism degree = 4** to deploy **2 model replicas** per instance.
- **E.** Split the model across all 8 GPUs using tensor parallelism degree = 8.

<details>
<summary>📖 View analysis & answer</summary>

### 🎯 What concept this tests
Optimizing GPU VRAM in continuous batching, and the tensor-parallelism division. Domain 4 (Operational Efficiency).

### 🔍 Core analysis — two diseases of waste
**Disease 1 — "reserving seats nobody sits in" (Sequence Length):** in an LLM server (DJL/vLLM), VRAM for the **KV Cache** is locked based on *Max Sequence Length*. Configure for 10,000 tokens → the GPU reserves VRAM for all 10,000. But customers actually use 1,000 → the surplus VRAM sits **empty**, when it could serve more customers concurrently (higher batch size/concurrency).

**Disease 2 — "a big oven baking a small cake" (Tensor Parallelism):** you rent a fancy 8-GPU EC2, but the model only needs 4 GPUs. By default the model runs on 4 GPUs and the **other 4 sit idle**.

**The fix:** (1) lower the max length to match reality → free VRAM → raise batch size (pack in more customers). (2) use all 8 GPUs by **replicating** the model: 8 GPUs ÷ 4 GPUs/model = **2 replicas** on the same machine → double the throughput.

### 🧩 Option-by-option breakdown
- **B — CORRECT.** DJL manages memory strictly. When real usage is far smaller (10×), reducing Max Sequence Length is the precise move: freeing VRAM from "seat reservations" lets it process more concurrent streams (higher rolling batch size).
- **D — CORRECT.** Model fits 4 GPUs → set **TP = 4**; machine has 8 GPUs → DJL auto-creates **2 replicas** (4×2=8). One machine with the throughput of two.
- **A — WRONG (burns money).** "Buy more instances" is exactly the cost increase the prompt wants to avoid; it doesn't fix the wasted resources inside the current machine.
- **C — WRONG (treats the wrong disease).** Speculative decoding speeds up **token generation (lower latency)** but doesn't save VRAM or raise utilization — it even **consumes extra VRAM** running a draft model in parallel.
- **E — WRONG (pointless stretching).** TP = 8 forces a model that needs only 4 GPUs to spread across all 8; the machine still has **just 1 replica**, throughput doesn't rise and may even drop due to 8 GPUs constantly communicating (overhead) → wastes the chance for a second replica.

### ✅ Conclusion: **B + D.**

### 💡 Exam tip
1. **Tensor Parallelism:** always do the division — EC2 has **X** GPUs, model fits **Y** GPUs → best config is **TP = Y**, replicas = **X / Y**. Don't spread TP = X unless truly needed.
2. **Continuous Batching / VRAM / Sequence Length** is a **"seesaw":** lower Max Sequence Length → free VRAM → higher Max Batch Size → serve more customers concurrently.

</details>

---

## Question 8 — Stream real-time suggestions to a web editor

> A familiar UX: how do you make the AI **"type character by character" (streaming)** onto the screen instantly, instead of making users stare at a spinner for ten seconds?

A law firm builds a web editor: when a lawyer clicks "analyze", the system must **immediately begin** streaming clause-revision suggestions (typewriter effect) through the interface. It uses a Bedrock FM. Requirement: **least operational overhead**.

Which architecture?

- **A.** SQS ingests articles → Step Functions process → Lambda → DynamoDB → API Gateway WebSocket streams back.
- **B.** **API Gateway WebSocket** linked to **Lambda**; the Lambda reads metadata to route the model, uses **Bedrock Prompt Management** to enforce style rules, and the **Bedrock streaming API** to return real-time.
- **C.** API Gateway **REST API** + Lambda function URLs, streaming via chunked transfer encoding.
- **D.** Application Load Balancer + ECS (custom containers) running routing logic + Bedrock streaming, WebSocket back to the web.

<details>
<summary>📖 View analysis & answer</summary>

### 🎯 What concept this tests
A **real-time response-streaming to web UI** architecture, fully serverless. Domain 2.

### 🔍 Core analysis — "the AI typing live"
With ordinary HTTP (send request → wait for the whole thing → return one block), UX is terrible. For the "typewriter" effect like ChatGPT, the web must keep a **"continuously open pipe"** to the server: as each token is produced, the server pushes it straight to the screen.

**The cure — WebSocket + Bedrock Streaming API:**
1. **WebSocket (two-way pipe):** API Gateway supports a WebSocket API, keeping the browser and server connected continuously, uninterrupted.
2. **Bedrock Streaming API:** instead of plain `InvokeModel`, use `InvokeModelWithResponseStream` — whatever token the AI produces is pushed immediately into the WebSocket pipe to the lawyer's screen. Fast, smooth, no lag.

### 🧩 Option-by-option breakdown
- **B — CORRECT (gold standard).** API Gateway WebSocket + Lambda + Bedrock Streaming API is best practice for real-time chatbots/AI assistants. Everything is serverless → "least operational overhead". The Lambda acts as a lightning-fast traffic cop reading metadata then routing to the right prompt; Prompt Management governs style.
- **A — WRONG (the queue strangles real-time).** Putting **SQS** at the front is fatal: SQS is **asynchronous** — you must "wait" (poll) to retrieve messages → adds latency, completely breaks the "immediate feedback" the lawyer needs on click.
- **C — WRONG (wrong protocol).** REST API is for **short-lived, synchronous** connections (one request - one response, then close); it can't hold a connection open to stream chunks smoothly like WebSocket. An AI task taking tens of seconds easily **times out**.
- **D — WRONG (self-inflicted overhead).** Standing up an ECS cluster, writing Dockerfiles, and managing containers just to run a tiny routing logic is **using a sledgehammer to crack a nut** — high overhead, against the requirement.

### ✅ Conclusion: **B.**

### 💡 Exam tip
1. **"Real-time feedback / typewriter effect / streaming to web UI"** → the winning pair is **API Gateway WebSocket + Bedrock Streaming API**.
2. **Drop SQS from real-time UI:** SQS is great for background processing, but never for a user clicking and waiting for a result on screen.
3. **Serverless > Containers:** with "LEAST operational overhead", prefer Lambda/API Gateway and confidently cross out options requiring self-managed EC2/ECS.

</details>

---

## Question 9 — Prompt governance + long-term compliance logging (Select TWO)

> Two survival requirements of finance: **managing human approval workflows (governance)** and **storing tamper-proof logs for auditors (compliance/audit)**.

An insurance company uses Bedrock for a customer-support assistant across business units. It needs: (1) prompt templates governed via **approval workflows**; (2) **comprehensive logging of all model invocations** with **7-year retention** for regulatory compliance. Requirement: **minimal operational overhead**.

Which two steps? (Select TWO.)

- **A.** Use **Amazon Bedrock Prompt Management** with multi-stage approval workflows.
- **B.** Store prompts in DynamoDB and self-write IAM + item-level permissions to control approvals.
- **C.** EventBridge captures invocation events → CloudWatch Logs → export to S3, enable S3 Object Lock for 7 years.
- **D.** Enable CloudTrail data events for all Bedrock APIs, deliver to CloudTrail Lake with 7-year retention.
- **E.** Enable **Bedrock model invocation logging** to S3, enable **S3 Object Lock (compliance mode) for 7 years**, create a prefix per business unit.

<details>
<summary>📖 View analysis & answer</summary>

### 🎯 What concept this tests
Two pieces: (1) prompt governance via a managed feature; (2) storing chat **content** immutably for 7 years. Domain 3.

### 🔍 Core analysis — two clear halves
**Half 1 — managing & approving prompts:** in a large company, developers can't just edit a prompt and ship it to customers. There must be a workflow: draft → manager approval → production. AWS has a "first-party" feature for this.

**Half 2 — logging for financial auditors:** auditors don't just want "who called the AI when", they want **exactly what the customer chatted and what the AI replied (full payload)**. And the log file must be **"locked down" for 7 years**, with no one (not even the IT director, not even the Root account) able to delete/modify it.

### 🧩 Option-by-option breakdown
- **A — CORRECT (half 1).** Amazon Bedrock Prompt Management is built-in: it provides the UI and flow to create → parameterize → version → **approval workflow**. Using the ready-made feature minimizes effort versus building your own.
- **E — CORRECT (half 2).** **Model invocation logging** records the **full** prompt + response straight to S3. **S3 Object Lock in Compliance mode** is the highest protection (WORM — Write Once Read Many): once locked, **no power on AWS** can delete/overwrite before the 7-year term — the mandatory standard for financial regulations (SEC/FINRA).
- **B — WRONG (reinventing the wheel).** Storing prompts in DynamoDB is fine, but you'd have to **build a management web app + write the approval-workflow logic** → huge, unnecessary overhead when A exists.
- **C — WRONG (a roundabout path prone to data loss).** The EventBridge → CloudWatch → export-to-S3 chain is bulky, has many breakpoints, and **doesn't guarantee capturing full payloads** of long chats → violates "minimal overhead".
- **D — WRONG (the classic CloudTrail trap).** CloudTrail is like a security camera at the door: it only records **metadata** (User John called `InvokeModel` at 10:00). It **never stores prompt/response content**. To a financial auditor, not knowing what the employee asked the AI is **worthless**.

### ✅ Conclusion: **A + E.**

### 💡 Exam tip
1. **"Prompt versions / approval workflows"** → pick **Amazon Bedrock Prompt Management**.
2. **"Regulatory compliance / cannot be deleted / N-year retention"** → the ultimate weapon is **Amazon S3 Object Lock (Compliance mode)**.
3. **Distinguish AI logs clearly:** need **who called an API** → CloudTrail; need **chat content (prompt/response payload)** → **Bedrock Model Invocation Logging**. Never pick CloudTrail when the prompt asks to audit **content**.

</details>

---

## Question 10 — Deploy a Python agent to AgentCore Runtime (Select TWO)

> Once you have slick Python code for an AI agent, how do you turn it into a proper service on Bedrock without playing **"construction worker"** (set up servers, package Docker, handle security yourself)?

A fintech company must deploy **existing Python agent code** to **Amazon Bedrock AgentCore Runtime**, aiming to **minimize infrastructure management**. The agent must handle both quick lookups (sub-second) and long reports (streaming over several minutes). The solution must **automatically** manage HTTP server, endpoint routing, and health monitoring.

Which two approaches with **minimal overhead**? (Select TWO.)

- **A.** Build a FastAPI server configuring `/invocations`, `/ping` endpoints and orchestrate containers yourself.
- **B.** Use the **AgentCore SDK** with the **`@app.entrypoint`** decorator to auto-handle server + endpoint setup.
- **C.** Deploy on ECS Fargate using a custom container running the AgentCore SDK.
- **D.** Deploy on a SageMaker AI real-time endpoint using a custom inference container.
- **E.** Use the **AgentCore starter toolkit** to automate packaging, containerization, and deployment.

<details>
<summary>📖 View analysis & answer</summary>

### 🎯 What concept this tests
The AgentCore toolset for getting code onto Runtime with the least infrastructure. Domain 2.

### 🔍 Core analysis — the "little code, lots of server-building" disease
For Bedrock to talk to your Python file, it must become an **HTTP server** opening a port, with `/ping` for health and `/invocations` for requests. Forcing an AI developer to write these servers, package Docker, and push to ECR is **exhausting**.

**The cure — AgentCore SDK & Starter Toolkit** rescue developers at two levels:
1. **Code level (`@app.entrypoint`):** instead of writing a verbose FastAPI server, just add **one line** `@app.entrypoint` above your Python function. AWS auto-wraps it into a proper server supporting both fast responses and long streaming.
2. **Deployment level (starter toolkit):** instead of clunky `docker build`/`docker push`, the toolkit **auto-packages your code, containerizes it, and deploys** to Bedrock.

### 🧩 Option-by-option breakdown
- **B — CORRECT (code level).** The `@app.entrypoint` decorator is the "secret weapon" satisfying "automatically manage HTTP server", removing the need to code a web framework. It smoothly supports both fast JSON (sub-second) and streaming (minutes).
- **E — CORRECT (infra level).** The starter toolkit handles the tedious infrastructure (Dockerfile, ECR, AgentRuntime) → the shortest, lowest-effort path (minimal overhead).
- **A — WRONG (reinventing the wheel).** Self-writing FastAPI means managing ports, writing `/ping` and `/invocations` logic, handling streaming yourself → overhead spikes.
- **C, D — WRONG (wrong platform).** Running code on **ECS Fargate** (C) or a **SageMaker custom container** (D) both force you to manage Docker lifecycle, define Tasks, configure Load Balancers/Auto Scaling. Meanwhile the prompt specified the target is **AgentCore Runtime** — detouring is both wrong and laborious.

### ✅ Conclusion: **B + E.**

### 💡 Exam tip
1. **AgentCore Runtime** + "Minimal overhead / No manual server config" → hunt two keywords: **`@app.entrypoint`** (for code) and **AgentCore starter toolkit** (for deployment).
2. If the prompt asks for the **fastest, easiest** way → confidently cross out options requiring a self-written web framework (FastAPI/Flask) or self-built custom containers.

</details>

---

## Question 11 — Source lineage for generated content (Select TWO)

> A **data lineage** problem in GenAI: when the AI generates content, how does a reviewer know **which source** it drew its knowledge from to verify accuracy?

An education publisher builds a practice-question generation system on Bedrock, using a mix of **curated** and **scraped** data. Reviewers must verify **credibility** by knowing **which source (source lineage)** the content came from. Requirement: **least operational overhead**.

Which two steps? (Select TWO.)

- **A.** Enable Bedrock invocation logging and self-build a system to correlate logs with the data source.
- **B.** **Tag FM outputs with source-data metadata.**
- **C.** **Register** the datasets (curated + scraped) with **AWS Glue Data Catalog**.
- **D.** Use SageMaker Clarify to explain model predictions.
- **E.** Use CloudTrail to log reviewer approval actions.

<details>
<summary>📖 View analysis & answer</summary>

### 🎯 What concept this tests
Source lineage = stick a **"source label"** on the output + keep a central **"source registry"**. Domain 1 & 3.

### 🔍 Core analysis — the "origin label" problem
Picture the AI system as a bakery sourcing ingredients from two places: a **certified farm** (curated) and a **flea market** (scraped). The reviewer (a food-safety inspector) looks at the finished cake (the AI-generated question) and asks: *"Which flour did this use?"*. To answer without ransacking the whole bakery, you need two things:
1. **Label the cake:** as it leaves the oven, attach a **metadata tag** with the batch ID.
2. **A source registry:** listing ID 1 = Farm A, ID 2 = Market B. The inspector reads the tag, checks the registry, done.

### 🧩 Option-by-option breakdown
- **B — CORRECT (labeling).** Program the system so each AI output auto-embeds the **source document's metadata**. Reviewers see the tag immediately, e.g. `Source: Wikipedia_Biology_p42`. Direct, automatic, very light.
- **C — CORRECT (registry).** AWS Glue Data Catalog is the central "registry": it indexes and manages every source (curated + scraped). Seeing the tag from B, reviewers look it up in Data Catalog to check credibility. Being a **managed service** → meets "least overhead".
- **A — WRONG (manual, complex).** Invocation logging only records "someone sent prompt X, got output Y"; it **doesn't know which PDF the AI drew from**. Correlating logs with the corpus requires a self-built, very complex matching system — high effort.
- **D — WRONG (wrong tool).** SageMaker Clarify specializes in **bias** detection and **explainability** for classic ML (why the AI approved this loan). It does **not** trace source text (lineage) for GenAI.
- **E — WRONG (off-topic).** CloudTrail only tracks **human actions** on AWS (API calls): "user A clicked Approve at 3 PM". It's completely blind to the question's data source.

### ✅ Conclusion: **B + C.**

### 💡 Exam tip
1. **"Source lineage / data origin / traceability"** → the pair **Metadata Tagging** (on the output) + **AWS Glue Data Catalog** (central metadata store).
2. **Don't confuse Explainability with Lineage:** Clarify explains **HOW** the model thinks; Data Catalog explains **WHERE** the model got its input data.

</details>

---

## Question 12 — RAG "silent failure" after a code update

> Playing **"AI detective"** to troubleshoot RAG: the system shows no red errors, everything looks perfect, but the results are **completely wrong**!

A financial-services company runs RAG: Bedrock for the embedding model, OpenSearch as the vector store, Lambda for embedding + search logic. **After a Lambda code update**, the app starts returning "no relevant information found" even for questions it answered well before. CloudWatch **shows no errors**, X-Ray confirms **successful FM invocation**, OpenSearch is healthy, latency is normal.

What's the cause?

- **A.** The document vectors in OpenSearch were deleted during the update and not re-indexed.
- **B.** The Lambda's IAM role is missing `bedrock:InvokeModel` permission.
- **C.** The FM temperature parameter was increased.
- **D.** The updated Lambda uses **a different version of the embedding model**.

<details>
<summary>📖 View analysis & answer</summary>

### 🎯 What concept this tests
**Embedding drift / vector space mismatch** — and the skill of reading the "crime scene" to rule out options. Domain 5.

### 🔍 Core analysis — the "speaking different languages" problem
Imagine all documents were originally "translated to **Japanese**" (embedding model V1) and stored in the OpenSearch library. Today a developer updates the Lambda code and accidentally switches to a "**French**" model (V2). When a user asks, the Lambda translates the question to "French" and searches a library that's all "Japanese" for similarity.

Technically, **nothing errors**: Lambda runs fine, OpenSearch is healthy, no error logs. But French and Japanese are in **different mathematical coordinate systems** → OpenSearch returns 0 results → "no relevant information found". This is **Embedding Drift**.

### 🧩 Option-by-option breakdown
- **D — CORRECT.** Switching embedding versions (e.g. `titan-embed-text-v1` → `v2`) creates a **completely different vector space**. The new code turns the query into new coordinates that can't match documents embedded with the old model. The failure is **"silent"** so there are no error logs (matches the clean CloudWatch clue).
- **A — WRONG.** A **Lambda code** update can't auto-reach into OpenSearch and wipe data; and if the DB were empty there'd usually be empty-query warnings. The problem centers on the code-update flow.
- **B — WRONG (contradicts the clues).** The prompt clearly states **"X-Ray confirms successful FM invocation"**. With missing IAM permission, X-Ray/CloudWatch would be **flooded with `AccessDeniedException`** and the app would crash, not gently return "no info found".
- **C — WRONG (misreads Temperature).** Temperature only affects **step 2 (Generation)**. But "no relevant information found" proves failure at **step 1 (Retrieval)**; the LLM found no source docs so it gives up. Temperature is unrelated to OpenSearch's search ability.

### ✅ Conclusion: **D.**

### 💡 Exam tip
1. **Silent failure in RAG:** the prompt describes *system was fine → updated code → everything still "green" (no errors, good latency) → but search returns empty/wrong* → almost certainly a **changed embedding model version**.
2. **Golden rule:** model A's vector space is **never** compatible with model B's (even v1 vs v2 from the same vendor). Change the embedding model → you **must re-index** the entire database.

</details>

---

## Question 13 — Monitor document ingestion into a Knowledge Base

> A platform engineer's task: when you throw 1,000 files at Bedrock to read, how do you know **which succeeded and which failed** (password-encrypted, too large, corrupt PDF)?

A company runs a Q&A application using a Bedrock Knowledge Base ingesting documents from multiple S3 buckets. It needs to **monitor the ingestion process** to find and troubleshoot **which documents failed processing**.

Which solution?

- **A.** Configure **knowledge base logging** to **CloudWatch Logs**, use **CloudWatch Logs Insights** to query failed documents.
- **B.** Enable CloudWatch Application Signals to auto-detect and alert on KB performance issues.
- **C.** Enable CloudTrail to track all APIs related to KB and ingestion.
- **D.** Implement Bedrock model invocation logging to capture detailed metrics about document processing and embedding.

<details>
<summary>📖 View analysis & answer</summary>

### 🎯 What concept this tests
Distinguishing the three Bedrock log types; picking **Knowledge Base logging** for document-level ingestion tracking. Domain 5.

### 🔍 Core analysis — the "inspecting incoming goods" problem
When the KB ingests from S3 it runs an **"Ingestion Job"**: open file → read text → chunk → embed → index. Along the way, errors occur **per file** (corrupt PDF, password-protected file, image with no text). You need a detailed report: *File A - Success, File B - Failed (reason), File C - Ignored*.

**The cure — Knowledge Base logging:** AWS has a **KB-specific** logging feature that records each document's fate (`SUCCESS`, `RESOURCE_IGNORED`, `EMBEDDING_FAILED`, `INDEXING_FAILED`) and ships it to CloudWatch Logs. With thousands of log lines, use **CloudWatch Logs Insights** (like SQL for logs) to filter instantly: *"give me all `INDEXING_FAILED` files"*.

### 🧩 Option-by-option breakdown
- **A — CORRECT.** KB logging is built **precisely** for this: ship logs to CloudWatch Logs to track **per-file** status; combine with Logs Insights to query and root-cause troubleshoot fast.
- **B — WRONG (wrong purpose).** CloudWatch Application Signals is APM — tracking microservices, which API is bottlenecked (on EKS/API Gateway). It does **not** read KB ingestion logs.
- **C — WRONG (familiar trap).** CloudTrail only records **API actions** (audit): "user A called `StartIngestionJob` at 10:00". But it **doesn't know** how many files are in the bucket or whether `report.pdf` succeeded — no **document-level** depth.
- **D — WRONG (wrong log type).** Model invocation logging only records content during **chat (inference)** — prompts & responses. Entirely unrelated to the KB's background **ingestion** phase.

### ✅ Conclusion: **A.**

### 💡 Exam tip (3 Bedrock log types — don't get fooled)
1. Need **who called an API, when** → **AWS CloudTrail**.
2. Need **what the customer chatted and how the AI replied (input/output payload)** → **Bedrock Model Invocation Logging**.
3. Need **which file failed during RAG ingestion (ingestion status)** → **Knowledge Base Logging + CloudWatch Logs Insights**.

</details>

---

## Question 14 — Maximize productivity with Amazon Q Developer (Select TWO)

> How do you fully leverage the **"AI coding assistant"** — both coding faster and automating testing — without **slowing** the workflow with manual checkpoints?

A cross-functional team building a GenAI app wants to **optimize developer productivity**, enforce consistent integration patterns, **automate performance tuning**, and **accelerate AI testing**. The team uses Amazon Q Developer.

Which two steps? (Select TWO.)

- **A.** Use Q Developer for security analysis but **require all code changes to be manually approved** by the security team before integration.
- **B.** Use Q Developer to **retrospectively analyze** and document patterns after the code is already written.
- **C.** Configure Q Developer to **auto-generate/refactor integration code and suggest performance optimizations** for AI components, applied across the modular codebase.
- **D.** Use Q Developer to fix issues during merge requests, but reserve most refactoring/optimization for **periodic manual review cycles**.
- **E.** Integrate Q Developer's **automated unit/integration test generation** into the **CI/CD pipeline**.

<details>
<summary>📖 View analysis & answer</summary>

### 🎯 What concept this tests
Using Q Developer **proactively + automatically**, not passively or with manual bottlenecks. Domain 2 & 4.

### 🔍 Core analysis — "maximize the AI assistant"
Amazon Q Developer isn't a trivia chatbot — it's an **AI coding companion** living inside the IDE and the DevOps (CI/CD) pipeline. The prompt asks three hard things: (1) optimize productivity (code faster, cleaner); (2) automate performance tuning; (3) accelerate testing. To achieve them, use Q Developer **proactively + automatically**, and **don't** create manual bottlenecks.

### 🧩 Option-by-option breakdown
- **C — CORRECT (productivity & performance).** It describes using Q Developer comprehensively and proactively while writing code: auto-generate, refactor for cleanliness, and crucially **suggest performance optimizations** — matching "automate performance tuning".
- **E — CORRECT (testing).** Wiring Q Developer's **auto test generation** into CI/CD (CodePipeline/GitHub Actions): every new push has the AI write and run tests → fully automatic, matching "accelerate AI testing".
- **A — WRONG (human bottleneck).** Security scanning is good, but "**all changes manually approved**" creates a huge bottleneck, killing "accelerate workflows".
- **B — WRONG (late, passive).** "Retrospective" = wait until the code is done, then bring out Q Developer to re-read + document — a passive "firefighting" approach that doesn't help write code faster from the start.
- **D — WRONG (loses the AI's value).** Telling AI to only catch trivial bugs while leaving refactoring/optimization to periodic manual cycles defeats the purpose of using AI to free up labor. Why have AI optimization (like C) and not use it?

### ✅ Conclusion: **C + E.**

### 💡 Exam tip
1. **Scoring keywords:** with Q Developer + a "productivity / accelerate" goal → choose options about **Automated, Generate, Refactor, Test generation in CI/CD**.
2. **Deadly keywords:** immediately cross out options containing **Manually approved**, **Manual review cycles**, or **Retrospectively**. AI is meant to run **automatically and in real time**.

</details>

---

## Question 15 — Choose the right SageMaker inference type for image generation

> Choosing the right **"delivery vehicle"**: SageMaker has 4 inference types, each with its own traits. Which one does a heavy image-generation model need to avoid "overload" or "timeout"?

A developer deploys a text-to-image model (tested via JumpStart). It must serve users generating images **on demand**, with three conditions: **use GPUs**, **payloads up to 40 MB**, and **responses within 10 minutes**.

Which strategy?

- **A.** **SageMaker Asynchronous Inference** on an accelerated-computing (GPU) instance; Lambda invokes the endpoint on demand.
- **B.** SageMaker Serverless Inference on a general-purpose instance; Lambda invokes the endpoint.
- **C.** SageMaker Real-Time Inference on a GPU instance; Lambda invokes the endpoint.
- **D.** SageMaker batch transform job on a GPU instance; Lambda starts the job.

<details>
<summary>📖 View analysis & answer</summary>

### 🎯 What concept this tests
Choosing a SageMaker inference type by **large payload + long processing + GPU needed + on-demand**. Domain 2 & 4.

### 🔍 Core analysis — matching three strict requirements
- **Needs GPU** → mandatory **Accelerated Computing** instance.
- **40 MB payload** → very heavy (ordinary API calls allow only a few MB).
- **10-minute wait** → very long (ordinary APIs time out at ~60 seconds).
- **On-demand** → user clicks and it runs, not batched at end of day.

**The cure — Asynchronous Inference:** built for very heavy models (high-res image generation). It allows **huge payloads (up to ~1 GB)** — plenty for 40 MB — and lets the model "think" for **a long time** — plenty for 10 minutes. It also has an **internal queue** to smoothly manage on-demand requests.

### 🧩 Option-by-option breakdown
- **A — CORRECT.** Asynchronous Inference supports GPU instances (accelerated computing), large payloads, and long processing, with an internal queue for on-demand → fits both 40 MB and 10 minutes.
- **B — WRONG (no GPU).** Serverless Inference is convenient (no servers to manage) but its **fatal weakness is no GPU support**. Image generation without GPU is infeasible or glacial.
- **C — WRONG (overload & timeout).** Real-Time is designed for **ultra-fast (millisecond)** tasks, so it's tightly limited: max payload **~6 MB** (prompt needs 40 MB) and timeout **~60 seconds** (prompt needs 10 minutes) → errors immediately.
- **D — WRONG (offline, doesn't fit).** Batch transform is **offline batch processing** — gather files at end of day, run one batch. Not designed for interactive apps where the user clicks and wants the image now (on-demand).

### ✅ Conclusion: **A.**

### 💡 Exam tip (SageMaker inference cheat sheet)
- **Real-time:** millisecond responses, small payload (~6 MB), short timeout (~60s) → live chatbots.
- **Asynchronous:** huge payload (~1 GB), long processing, built-in queue → **image generation (computer vision), video processing**.
- **Serverless:** no server management, spiky traffic, but **no GPU support**.
- **Batch Transform:** offline processing, analyze a big chunk of data overnight, not for interactive apps.

</details>

---

## Question 16 — Large-scale vector search, infrequent, cheapest

> A cost-optimization problem of the **"raise an army for years, use it for one hour"** kind: store a massive amount of data and **search only occasionally**, without burning money on server upkeep.

An Earth-observation application needs **similarity search across 80 million satellite images**, ingests new images daily, but performs searches **infrequently** — only when analysts need comparisons. Requirement: **cheapest**, responsive search, **no infrastructure management**.

Which solution?

- **A.** Store vectors in OpenSearch Serverless, use vector search.
- **B.** Store vectors in DynamoDB, self-implement similarity logic via Lambda.
- **C.** Create an **Amazon S3 vector bucket** with vector indexes to store embeddings and perform similarity search.
- **D.** Store vectors in RDS for PostgreSQL + pgvector.

<details>
<summary>📖 View analysis & answer</summary>

### 🎯 What concept this tests
Choosing the **cheapest** vector store for large volume but **sparse queries** — not an always-on store charging upkeep. Domain 1 & 4.

### 🔍 Core analysis — the killer keyword "infrequently"
- If you rent an **always-on database** (RDS) or a system that must **maintain background compute** (OpenSearch Serverless), you pay monthly even when nobody searches all week → very wasteful.
- Second requirement: **no infrastructure management** — no choosing RAM/CPU, no server config.

**The cure — Amazon S3 Vectors:** a **truly serverless** S3 feature. Throw millions of vectors in; when no one searches, you pay only **dirt-cheap** S3 storage. When an analyst clicks "Search", AWS charges just for that click (**pay-as-you-go**) → perfect for **low frequency + cheapest cost**.

### 🧩 Option-by-option breakdown
- **C — CORRECT.** S3 Vectors is managed (no infra to manage), scales to **billions of vectors**. Thanks to S3's **pay-as-you-go** nature, it's especially cost-effective for **infrequent searches** — you don't pay to "feed" a server while idle.
- **A — WRONG (base-capacity charge).** OpenSearch Serverless has powerful k-NN vector search but is designed for **continuous, high-frequency** search. Despite "serverless", it still charges **minimum OCUs (OpenSearch Compute Units)** to stay ready → wasteful background OCU cost when searches are rare.
- **B — WRONG (wrong tool).** DynamoDB is a great Key-Value store but has **no built-in vector search**. Forcing Lambda to scan 80 million records and self-compute distances → glacial and huge Lambda compute cost.
- **D — WRONG (must manage server + upkeep).** RDS + pgvector requires **provisioning** a DB server (db.m5.large…), paying **hourly even when idle** + maintenance → violates both requirements.

### ✅ Conclusion: **C.**

### 💡 Exam tip
1. **"Infrequently" + "Cost-effective" + "Vector Search"** → find the cheapest storage with vector support: **Amazon S3 Vectors**.
2. **Rule out always-on databases:** confidently cross out RDS, Aurora, OpenSearch (even Serverless) if the prompt stresses "only occasional users". These are cost-effective only with **continuous / high-frequency** traffic.

</details>

---

## Question 17 — Identify which guardrail rule blocked the request

> Playing **"AI investigator"**: when the system refuses a customer, how do you know exactly **why** it blocked (profanity, forbidden topic, or a leaked ID number)?

An AI assistant on Bedrock has multiple guardrails (prompt injection, sensitive-information filtering, denied-topic blocking). When a query is blocked, the developer needs a **detailed analysis of which specific guardrail rule fired** to fine-tune and distinguish legitimate queries from real threats.

Which configuration gives the most detailed analysis?

- **A.** Enable Bedrock model evaluation with automated evaluation jobs + guardrail assessment metrics.
- **B.** Enable model invocation logging, set CloudWatch alarms on `InvocationsIntervened` filtered by `GuardrailContentSource`.
- **C.** Configure **guardrail tracing** (`{"trace": "enabled"}`), monitor `InvocationsIntervened` filtered by **`GuardrailContentSource`**.
- **D.** Configure **guardrail tracing** (`{"trace": "enabled"}`), monitor `InvocationsIntervened` filtered by **`GuardrailPolicyType`** (ContentPolicy, TopicPolicy, SensitiveInformationPolicy).

<details>
<summary>📖 View analysis & answer</summary>

### 🎯 What concept this tests
Troubleshooting/monitoring Guardrails: enable **trace** + filter by the right **dimension** to learn **which rule** intervened. Domain 3.

### 🔍 Core analysis — the "find the culprit" problem
Bedrock Guardrails is like a **multi-layer checkpoint** (several different "bouncers"):
1. The bouncer screening **"toxic words"** (Content Policy).
2. The bouncer screening **"forbidden topics — e.g. politics"** (Topic Policy).
3. The bouncer screening **"leaked PII"** (Sensitive Information Policy).

When a customer is "kicked out" (blocked), you need to know exactly **which bouncer acted**. To do that: (1) enable **"tracing"** to record the moderation log; (2) filter the log by **Policy Type** to know the exact name of the layer that intervened.

### 🧩 Option-by-option breakdown
- **D — CORRECT.** `{"trace": "enabled"}` is required to make the system emit a detailed report. Crucially, filtering by **`GuardrailPolicyType`** tells you exactly **which rule was violated** (ContentPolicy / TopicPolicy / SensitiveInformationPolicy) → so the developer knows what to fine-tune.
- **C — WRONG (wrong focus).** It also enables tracing but filters by **`GuardrailContentSource`** — that only tells you whether the block happened on **input (question)** or **output (answer)**, **not which rule** was violated.
- **B — WRONG (missing tracing).** Model invocation logging records input/output but **doesn't provide the detailed JSON structure** of the guardrail's internal decision; and it filters by the wrong dimension like C.
- **A — WRONG (wrong tool, wrong time).** Model Evaluation is for the **testing/pre-production** phase to score model intelligence, **not** a tool for monitoring/troubleshooting **real-time** production queries.

### ✅ Conclusion: **D.**

### 💡 Exam tip
1. To know **which rule** the system blocked on → `GuardrailPolicyType` (Topic / Content / PII).
2. To know whether it blocked the **question or the answer** → `GuardrailContentSource` (Input / Output).
3. Always need **`{"trace": "enabled"}`** in the Converse API config to get detailed analysis.

</details>

---

## Question 18 — Secure auth, IdP federation, no long-lived credentials (Select TWO)

> Critical enterprise knowledge: let staff/apps use AI securely **without risking leaked passwords or access keys** — via a **"single-use gate pass"**.

A company needs secure authentication for a third-party application that uses Bedrock. The solution must: integrate with the **existing IdP**, keep **comprehensive audit logs**, **eliminate long-lived credentials**, and provide **temporary access** to Bedrock.

Which two solutions? (Select TWO.)

- **A.** Integrate **OIDC with Amazon Cognito**: authenticate via the IdP then exchange tokens for temporary AWS credentials to access Bedrock.
- **B.** An API Gateway Lambda authorizer validates against LDAP then issues JWTs for Bedrock access.
- **C.** Create IAM users per employee, assign permissions via IAM policies, rotate with Secrets Manager.
- **D.** Create an IAM role + federation via STS AssumeRole, **store the application's IAM user credentials in the config file**.
- **E.** Deploy **AWS IAM Identity Center with SAML federation** to the IdP, configure permission sets granting Bedrock access.

<details>
<summary>📖 View analysis & answer</summary>

### 🎯 What concept this tests
**Identity Federation** to issue **temporary credentials** from an IdP, eliminating static keys. Domain 3.

### 🔍 Core analysis — the "single-use gate pass" problem
In cloud security, the most dangerous enemy is **long-lived credentials** (permanent Access Key & Secret Key). Create a permanent key and embed it in source code, and if a hacker steals the source → your system is compromised **forever**.

**The cure — Identity Federation:** the standard approach is to **not create internal AWS accounts**, but to connect AWS to your existing HR/identity system (IdP like Okta, Microsoft Entra ID). When an employee logs into the IdP successfully, AWS issues a **temporary credential** valid for only a few hours — it self-destructs when time's up. No static key is stored, so a stolen pass is useless.

### 🧩 Option-by-option breakdown
- **A — CORRECT (for the application).** Amazon Cognito manages sign-in for **applications (web/app)**, supporting **OIDC** to federate with the company IdP. The key win: Cognito Identity Pools **exchange the IdP token for temporary AWS credentials** to call Bedrock; every action is auto-logged by CloudTrail.
- **E — CORRECT (for the workforce).** **AWS IAM Identity Center** (formerly AWS SSO) is the ultimate tool for **SAML 2.0 federation** with the company identity system: it grants access entirely via **temporary credentials** (eliminating long-lived credentials), with all sign-ins/API calls logged in CloudTrail.
- **B — WRONG (reinventing the wheel, insecure).** Self-writing a Lambda to connect to legacy **LDAP** then issue **JWTs** is a risky custom solution; the Bedrock API **doesn't accept custom JWTs directly** — they still need exchanging for AWS credentials.
- **C — WRONG (violates the core requirement).** Creating **IAM Users** per person = creating **long-lived credentials** (permanent passwords & Access Keys) — an infamous anti-pattern, directly violating "eliminate long-lived credentials". Rotation via Secrets Manager doesn't change the static-key nature.
- **D — WRONG (serious security flaw).** The first half mentions STS AssumeRole (issuing temporary credentials) correctly, but the second half pushes **"store IAM user credentials in the config file"** — hardcoding static keys into the app is a **fatal flaw**, completely violating the requirement.

### ✅ Conclusion: **A + E.**

### 💡 Exam tip
1. **"Eliminate long-lived credentials"** → immediately cross out any option creating **IAM Users** or storing **Access Keys**.
2. **Two federation services issuing temporary credentials:** **AWS IAM Identity Center (SAML)** for the **internal workforce** accessing AWS; **Amazon Cognito (OIDC/SAML)** for **customer-facing applications**. Both are correct for this security pattern.

</details>

---

## Question 19 — Peak-hour throttling, same FM, cheapest

> The "disease" of over-successful systems: **peak-hour congestion (throttling)**. How do you expand the AI's serving limit without **paying a fortune** or writing your own load-balancing code?

An e-commerce platform uses Bedrock to generate product descriptions; the app lives in **one Region**. At peak it hits "Too many requests, please wait before trying again". It must **increase throughput** at peak **without additional operational overhead**, **maintain compatibility with the existing Bedrock API**, and **use the exact same FM**. Requirement: **cheapest**.

Which solution?

- **A.** A Lambda calls Bedrock with the original Region as default, failing over to a secondary Region on error.
- **B.** Use **Cross-Region Inference** to distribute traffic across multiple Regions within a geographic area.
- **C.** Use prompt routing to distribute traffic across multiple FMs in the same family.
- **D.** Use Provisioned Throughput to provision a higher throughput level for the FM.

<details>
<summary>📖 View analysis & answer</summary>

### 🎯 What concept this tests
Exceeding quota limits at peak with **Cross-Region Inference**, on-demand, no code changes. Domain 1 & 4.

### 🔍 Core analysis — the "peak-hour traffic jam" problem
Each Region (e.g. `us-east-1`) has a **quota** on API calls per minute. During a Flash Sale, requests spike, Bedrock in that Region congests → rate-limit / throttling errors.

**The cure — Cross-Region Inference:** instead of forcing developers to write complex "if Region A errors, call Region B" code, Bedrock provides this feature: just change the **Model ID** to an **Inference Profile ARN**, and AWS becomes the traffic cop — *"Region A is jammed, I'll auto-route your request to Region B (which is free), return the result, same price!"*. The system doubles its quota with **zero operational code**.

### 🧩 Option-by-option breakdown
- **B — CORRECT.** Cross-Region Inference satisfies every requirement: avoids throttling by pooling multiple Regions' quotas; still uses the **exact same FM**; **no** complex code/infra; charged by **actual tokens used (on-demand)** → **cheapest** for traffic spikes.
- **D — WRONG (the "big spender" option, very expensive).** Provisioned Throughput means **buying a block** of GPUs running all month. It fixes throttling but **costs a fortune** (tens of thousands of USD/month). For a company that only congests at **peak hours**, buying 24/7 servers is enormous waste, against "cost-effective".
- **A — WRONG (reinventing the wheel).** Self-writing a Lambda Try/Catch then failing over to another Region is exactly what people did **before** Cross-Region Inference existed; maintaining the code and tracking errors → adds **operational overhead**.
- **C — WRONG (violates a technical requirement).** Intelligent Prompt Routing routes by question **difficulty** (easy → Haiku for cheapness, hard → Sonnet). Its catch: it forces **multiple different FMs** → blatantly violates "must use the **same FM**".

### ✅ Conclusion: **B.**

### 💡 Exam tip
1. **"Too many requests / throttling limits / peak spikes" + "cost-effective"** → the solution is always **Cross-Region Inference** (elastically expands quota, pay-per-use).
2. **Provisioned Throughput:** only pick it when the prompt stresses **high, stable, continuous 24/7** load (consistent, predictable workload) with long-term commitment. With "spikes" or "peak periods", absolutely **don't** pick Provisioned Throughput.

</details>

---

## Question 20 — Redact PII before feeding it to search

> A sensitive finance/banking problem: **handling PII (Personally Identifiable Information)** before feeding it to AI. How do you **"scrub"** phone numbers, account numbers, and customer names from millions of old emails before the AI reads them?

A bank wants a mobile app to help users query account information, using a corpus of **emails** between customers and support staff (stored in S3) as the source. The data contains **PII that must not appear in search results**.

Which solution?

- **A.** Use Kendra to search the emails in S3, integrate a Bedrock FM, use a **system prompt** to remove PII during query processing.
- **B.** Use **Amazon Comprehend** to detect + **redact PII** from the emails in S3, integrate with **Amazon Kendra** to search the processed data.
- **C.** Use Textract to extract text, Macie to scan for PII in S3, integrate Kendra to search.
- **D.** Use Comprehend to redact PII, integrate **DocumentDB** for query-based search.

<details>
<summary>📖 View analysis & answer</summary>

### 🎯 What concept this tests
A **clean-first, search-second** architecture: redact PII (Comprehend) → index for search (Kendra). Domain 3.

### 🔍 Core analysis — the "data laundry" problem
Email records contain tons of sensitive data (names, card numbers, addresses). **Security rule #1** (especially in finance): **never feed raw, unclean data into a search or AI system**. You need a "washing machine" to **redact** all of it (e.g. "Account 123456" → "Account [ACCOUNT_NUMBER]").

**The cure — Comprehend + Kendra:**
1. **Amazon Comprehend (the washing machine):** an NLP service with **PII Redaction** — it reads millions of emails, auto-finds card numbers/names, and replaces them with `***` or anonymized labels.
2. **Amazon Kendra (the smart file cabinet):** after Comprehend "washes" the data, feed the clean data into Kendra — a powerful **semantic enterprise search** engine. Users search freely with zero risk of leaking others' PII.

### 🧩 Option-by-option breakdown
- **B — CORRECT.** Comprehend (detect + redact PII) + Kendra (index + enterprise search) is the perfect, secure, fully automated solution.
- **A — WRONG (a fatal security mistake).** Trusting a **system prompt** to protect PII is very risky: an LLM isn't a deterministic filter and can be **jailbroken** or hallucinate and leak info. Sensitive data must be **redacted BEFORE** the AI ever sees it.
- **C — WRONG (entirely wrong tools).** **Textract** is OCR for reading text from images/scanned PDFs — emails are already plain text, so Textract is pointless. **Macie** is an **auditing** tool: it only **alerts** "this bucket contains PII", it **does not redact** or transform files for AI.
- **D — WRONG (search in the wrong place).** The first half (Comprehend) is right, but the second half uses **DocumentDB** (a MongoDB-compatible JSON DB) as the search tool. DocumentDB has **no natural-language understanding** for users to chat-search like Kendra → wrong architecture.

### ✅ Conclusion: **B.**

### 💡 Exam tip (tool cheat sheet)
- **Amazon Comprehend** — NLP: the **only** tool that reads text and **redacts PII**.
- **Amazon Kendra** — semantic enterprise search: find internal docs via natural language.
- **Amazon Macie** — security discovery/assessment: only **alerts** on PII, **doesn't redact**.
- **Amazon Textract** — OCR for images/PDFs: **not** for raw text like emails.
- **Amazon DocumentDB** — NoSQL JSON: **not** a natural-language search engine.

</details>

---

> **End of 20 questions.** This is original content written by the author for concept practice, not real exam questions. See [DISCLAIMER](../../DISCLAIMER.md).
