# Practice Questions (Original)

[← Về Practice Exam](./README.md)

> 20 câu hỏi **gốc do tác giả tự viết**, viết lại từ *concept* cần kiểm tra (khác ngành, khác bối cảnh, khác số liệu so với bất kỳ đề nào). **Không phải đề thi thật, không copy AWS Skill Builder.** Xem [DISCLAIMER](../../DISCLAIMER.md).
>
> **Cách dùng:** đọc scenario → tự chọn đáp án trước → mở khối "Xem phân tích & đáp án". Phần **Phân tích cốt lõi**, **Mổ xẻ đáp án** và **Mẹo đi thi** quan trọng hơn việc đúng/sai.

---

## Câu 1 — RAG: kết quả tốt bị chìm xuống dưới (Chọn HAI)

> Câu này chạm tới một **"căn bệnh"** vô cùng phổ biến trong các hệ thống RAG (Retrieval-Augmented Generation — Truy xuất và Sinh văn bản): hệ thống tìm ra cả rổ tài liệu liên quan, nhưng tài liệu **ngon nhất lại chìm tận đáy**. Làm sao đẩy nó lên top 1 một cách **nhàn hạ nhất**?

Một nhà mạng viễn thông xây ứng dụng RAG trên Amazon Bedrock để tra cứu tài liệu xử lý sự cố mạng. Hệ thống truy xuất được nhiều đoạn liên quan, nhưng kỹ thuật viên phản ánh đoạn hữu ích nhất thường nằm ở cuối danh sách. Công ty muốn cải thiện thứ hạng kết quả với **công sức vận hành (operational overhead) tối thiểu**.

Hai bước nào đáp ứng yêu cầu? (Chọn HAI.)

- **A.** Bật **hybrid search** trên Knowledge Bases (kết hợp vector embeddings với khớp từ khóa) dùng OpenSearch Serverless làm backend.
- **B.** Tự cài plugin Learning to Rank (học xếp hạng) trên OpenSearch, thu thập dữ liệu click người dùng và huấn luyện mô hình chấm điểm.
- **C.** Dùng **reranker model (mô hình xếp hạng lại) có sẵn của Amazon Bedrock** để sắp xếp lại kết quả theo độ liên quan ngữ nghĩa.
- **D.** Tạo Aurora PostgreSQL + pgvector và tự viết thuật toán chấm điểm tương đồng tùy chỉnh.
- **E.** Dùng SageMaker JumpStart + Kendra Intelligent Ranking để tạo thuật toán chấm điểm riêng.

<details>
<summary>📖 Xem phân tích & đáp án</summary>

### 🎯 Câu này test concept gì
Cải thiện chất lượng **retrieval** (truy xuất) của RAG bằng **tính năng managed có sẵn**, thay vì tự xây. Thuộc Domain 1 (FM Integration & Data).

### 🔍 Phân tích cốt lõi — bài toán "đãi cát tìm vàng"
Khi RAG tìm trong kho dữ liệu, nó nhặt ra một rổ tài liệu (ví dụ 20 đoạn). Vấn đề là đoạn chứa câu trả lời xuất sắc nhất lại nằm ở vị trí số 19. Khi AI đọc từ trên xuống, nó bị **"quá tải thông tin"** và bỏ sót — hiện tượng kinh điển gọi là *"Lost in the middle"* (lạc giữa danh sách).

**Thuốc đặc trị — combo "Hybrid Search + Reranker":** AWS cung cấp sẵn 2 vũ khí để đẩy đoạn ngon nhất lên top 1 mà gần như không phải code:
1. **Hybrid Search (tìm kiếm lai):** thay vì chỉ tìm theo vector (ngữ nghĩa mờ ảo), kết hợp thêm khớp **keyword** (đúng từng chữ). Hai phương pháp bù trừ nhau, tự đẩy tài liệu chuẩn nhất lên trên. Trên Bedrock Knowledge Bases, đây chỉ là một **nút bật/tắt cấu hình** — cực nhàn.
2. **Reranker Model (mô hình xếp hạng lại):** đóng vai "giám khảo". Sau khi hệ thống tìm ra 20 đoạn, ném cả 20 cho reranker (ví dụ Cohere Rerank trên Bedrock). Giám khảo này chấm điểm lại và đảo thứ tự, kéo đoạn xuất sắc nhất lên top 1. Tính năng **built-in**, không cần tự train.

### 🧩 Mổ xẻ từng đáp án
- **A — ĐÚNG.** Chuyển từ vector thuần sang hybrid search là cách nhanh và hiệu quả nhất để tăng độ chính xác RAG. Cấu hình trên Knowledge Bases (backend OpenSearch Serverless) hoàn toàn là **bật tính năng có sẵn**, không viết thuật toán, rất nhàn.
- **C — ĐÚNG.** Bedrock reranker sinh ra **chính xác** cho bài toán "kết quả tốt nằm dưới"; nó tự reorder. Vì là managed service, đáp ứng tuyệt đối tiêu chí *minimal operational overhead* — chỉ việc gọi API.
- **B — SAI (quá vất vả).** Learning to Rank là plugin mã nguồn mở: bạn phải **tự thu thập dữ liệu click**, **tự huấn luyện** mô hình chấm điểm, và **tự bảo trì**. Overhead khổng lồ, vi phạm đề bài.
- **D — SAI (chế lại bánh xe).** Dùng pgvector thì được, nhưng bắt lập trình viên **"tự viết thuật toán chấm điểm tương đồng"** là cực hình: tự code toán học, tự quản hạ tầng — ngược hẳn "minimal overhead".
- **E — SAI (cồng kềnh).** Ghép SageMaker JumpStart + Kendra rồi lại "tạo thuật toán tùy chỉnh" là kiến trúc quá nặng nề, trong khi Bedrock đã có reranker sẵn bên trong.

### ✅ Kết luận: **A + C.**

### 💡 Bỏ túi mẹo đi thi
1. **"Improve ranking / most relevant lower / better retrieval / tăng độ chính xác RAG"** → cặp bài trùng vô địch trên AWS luôn là **Hybrid Search** (vector + keyword) và **Reranker Models** (chấm điểm + đảo vị trí).
2. **Từ khóa chết chóc "MINIMAL overhead":** gạch bỏ ngay mọi đáp án chứa *Create custom algorithm* (tự viết thuật toán), *Train custom model* (tự huấn luyện), *Learning to Rank plugin* (tự quản plugin). Luôn ưu tiên tính năng **built-in/managed** của Bedrock.

</details>

---

## Câu 2 — Đồng bộ Knowledge Base real-time và resilient

> Đây là một mẫu kiến trúc kinh điển: **xử lý sự kiện thời gian thực một cách bền bỉ** (event-driven & resilient). Làm sao để AI cập nhật tài liệu ngay lập tức mà **không bị sập** khi hàng loạt file thay đổi cùng lúc?

Một bệnh viện vận hành trợ lý AI trả lời dựa trên tài liệu quy trình nội bộ, dùng Amazon Bedrock Knowledge Bases với nguồn là Amazon S3. Tài liệu mới phải xuất hiện trong câu trả lời **càng sớm càng tốt**, tài liệu đã xóa phải bị loại **càng sớm càng tốt**. Giải pháp phải **scalable, event-driven, và resilient (bền bỉ/chịu lỗi)**.

Giải pháp nào đáp ứng?

- **A.** EventBridge Scheduler chạy Lambda mỗi 5 phút để dò thay đổi trong S3 rồi gọi API thêm/xóa tài liệu.
- **B.** S3 Event Notifications gọi trực tiếp Lambda khi có sự kiện tạo/xóa; Lambda gọi `IngestKnowledgeBaseDocuments` / `DeleteKnowledgeBaseDocuments`.
- **C.** S3 Event Notifications đẩy sự kiện tạo/xóa vào hàng đợi **Amazon SQS**; Lambda poll hàng đợi rồi gọi `IngestKnowledgeBaseDocuments` / `DeleteKnowledgeBaseDocuments`.
- **D.** EventBridge Scheduler chạy Lambda mỗi 5 phút gọi `StartIngestionJob` để đồng bộ lại toàn bộ.

<details>
<summary>📖 Xem phân tích & đáp án</summary>

### 🎯 Câu này test concept gì
Kiến trúc **event-driven + buffering (bộ đệm) bằng SQS** để vừa real-time vừa chịu được spike tải. Thuộc Domain 2 (Implementation & Integration).

### 🔍 Phân tích cốt lõi — bài toán "cập nhật siêu tốc nhưng không được sập"
Đề bài cài 3 từ khóa, mỗi từ loại bớt phương án:
1. **"Càng sớm càng tốt" (as soon as possible):** phải real-time, có file là cập nhật ngay → **loại bỏ mọi phương án đặt lịch (Scheduled/Cron)**.
2. **"Event-driven":** file thay đổi (sự kiện) tự kích hoạt hệ thống.
3. **"Resilient" — từ khóa ăn tiền nhất:** nếu bệnh viện upload 1.000 file trong một giây, Bedrock sẽ bị ngợp và **đánh rớt yêu cầu (throttling)**. Để bền bỉ, bắt buộc phải có một **"phễu/bộ đệm"** đứng giữa, chứa 1.000 file đó lại rồi rót từ từ cho Bedrock xử lý.

**Thuốc đặc trị — Amazon SQS:** hàng đợi SQS chính là chiếc phễu. Nó hứng mọi sự kiện từ S3, giữ an toàn không rơi mất dữ liệu, và có cơ chế **tự retry** nếu Lambda cập nhật Bedrock bị lỗi. Lambda thong thả lấy từng tin nhắn ra xử lý với tốc độ ổn định, không lo quá tải.

### 🧩 Mổ xẻ từng đáp án
- **C — ĐÚNG.** Chuỗi `S3 → SQS → Lambda → Bedrock API` là Best Practice của AWS: S3 Event Notifications cho tính "event-driven" và "real-time"; SQS cho tính "resilient" (bộ đệm chống sốc tải + tự retry); hệ thống scale thoải mái không lo sập.
- **A — SAI.** Chạy mỗi 5 phút vi phạm "càng sớm càng tốt" và không phải event-driven. Bắt Lambda **tự "dò thay đổi"** (so sánh file cũ/mới) cực tốn tài nguyên và kém hiệu quả.
- **B — SAI (chết vì thiếu bền bỉ).** `S3 → Lambda` trực tiếp cũng nhanh và event-driven, **nhưng thiếu bộ đệm**. Gặp spike tải, Bedrock throttling → Lambda fail → không có hàng đợi lưu lại sự kiện lỗi → file đó **không bao giờ được cập nhật** (mất dữ liệu).
- **D — SAI.** Vừa chậm (5 phút) vừa dùng sai API: `StartIngestionJob` **quét và đồng bộ lại toàn bộ kho** S3, chậm và đắt hơn nhiều so với chỉ thêm/xóa đúng file vừa đổi (incremental update qua `IngestKnowledgeBaseDocuments`).

### ✅ Kết luận: **C.**

### 💡 Bỏ túi mẹo đi thi
1. **"As soon as possible / near real-time" + "Resilient" / "Traffic spikes":** đáp án đúng gần như luôn có **Amazon SQS** làm bộ đệm (decoupling/buffering).
2. **Direct invocation (S3 → Lambda):** bỏ ngay nếu đề nhấn "resilient" hoặc "traffic spikes".
3. **Scheduler / Cron job:** gạch ngay nếu đề có "as soon as possible" hoặc "event-driven". Lên lịch là **kẻ thù của real-time**.

</details>

---

## Câu 3 — Phân tích ảnh/video với ít công sức nhất

> Bài toán cho AI **"có đôi mắt"**: phân tích hình ảnh và video, rồi tổng hợp xu hướng lên dashboard cho lãnh đạo — mà đội IT **không phải "bán mạng"** đi dán nhãn và huấn luyện model.

Một hãng truyền thông thể thao muốn phân tích **video và ảnh** từ các trận đấu công khai để hiểu các yếu tố chiến thuật và xu hướng, lưu thông tin trích xuất và dựng dashboard tóm tắt. Yêu cầu **ít công sức vận hành nhất (LEAST operational overhead)**.

Giải pháp nào đáp ứng?

- **A.** EventBridge kích hoạt Lambda dùng QuickSight Q để phân tích video/ảnh, lưu S3, dựng dashboard QuickSight.
- **B.** Dùng **Step Functions** điều phối xử lý video/ảnh bằng **multimodal FMs trên Amazon Bedrock**, lưu S3, trực quan hóa bằng **QuickSight**.
- **C.** Dùng Rekognition Custom Labels huấn luyện mô hình riêng, lưu DynamoDB, dashboard bằng Managed Grafana + plugin tùy chỉnh.
- **D.** Dùng Claude phân tích mô tả văn bản, dùng Stable Diffusion để "phân tích" ảnh, lưu OpenSearch, dashboard Grafana.

<details>
<summary>📖 Xem phân tích & đáp án</summary>

### 🎯 Câu này test concept gì
Cho AI hiểu nội dung ảnh/video bằng **multimodal Foundation Model có sẵn** (không tự train), kết hợp dashboard bằng dịch vụ managed. Thuộc Domain 1 & 2.

### 🔍 Phân tích cốt lõi — bài toán "AI có đôi mắt"
Đầu vào là **video và ảnh** (visual data). Để AI "xem" được, ta cần một bộ não có khả năng **đa phương thức (multimodal)**. Đồng thời, đề có từ khóa sát thủ: **LEAST operational overhead** — nghĩa là phải chọn dịch vụ có sẵn, **không tự huấn luyện**, không quản máy chủ, không tự code giao diện.

**Thuốc đặc trị — combo Bedrock Multimodal + QuickSight:**
1. **Multimodal FMs** (Claude, Amazon Nova Pro): vừa đọc chữ vừa "xem" được ảnh/video. Không cần huấn luyện thêm — nó tự hiểu cầu thủ đang chạy sơ đồ chiến thuật gì.
2. **Amazon QuickSight:** dịch vụ dựng dashboard (BI) kéo-thả, không cần máy chủ — quá nhàn cho một báo cáo xu hướng.

### 🧩 Mổ xẻ từng đáp án
- **B — ĐÚNG.** Bedrock multimodal FM (có sẵn) cho AI "xem" thẳng video mà không cần dán nhãn/huấn luyện; Step Functions điều phối quy trình trích xuất là Best Practice; QuickSight dựng dashboard ngay từ dữ liệu S3 (qua Athena) mà không cài phần mềm hay plugin phức tạp.
- **A — SAI (hiểu sai QuickSight Q).** QuickSight **Q** là tính năng hỏi đáp bằng ngôn ngữ tự nhiên trên **dữ liệu dạng bảng/số liệu** (ví dụ "doanh thu tháng này?"). Nó **không thể "xem"** file `.mp4`/`.jpg` để phân tích hình ảnh.
- **C — SAI (cực hình vận hành).** Rekognition Custom Labels = đội của bạn phải tự tải hàng ngàn ảnh, **tự vẽ bounding box**, tự dán nhãn, **tự huấn luyện**; rồi lại **tự viết plugin tùy chỉnh** cho Grafana. Công sức khổng lồ, vi phạm "least overhead".
- **D — SAI (sai công cụ nghiêm trọng).** **Stable Diffusion là mô hình sinh ảnh (text-to-image)** — dùng để *vẽ* ảnh mới, **không** dùng để *phân tích/nhận diện* ảnh có sẵn. Ngoài ra tự quản cụm OpenSearch cũng tốn nhiều công vận hành.

### ✅ Kết luận: **B.**

### 💡 Bỏ túi mẹo đi thi
1. **"Analyze videos and photos / image understanding / no custom training"** + LEAST overhead → chọn ngay **Multimodal FMs trên Bedrock** (Claude, Nova).
2. **Rekognition Custom Labels:** chỉ chọn khi đề cho rõ "công ty có tập ảnh gán nhãn riêng + muốn nhận diện logo/sản phẩm độc quyền". Nhận diện vật thể chung chung → dùng dịch vụ có sẵn.
3. **"Dashboard / Business Intelligence" một cách nhàn hạ (serverless/managed)** → **Amazon QuickSight** là câu trả lời mặc định.

</details>

---

## Câu 4 — Sắp xếp quy trình đánh giá thay model

> Dạng câu **sắp thứ tự (ordering)** với một kịch bản rất thực tế: làm sao **"thay tướng giữa trận"** — thay một model AI cũ đang phục vụ khách hàng bằng model mới — mà không làm hỏng hệ thống?

Một công ty muốn thay model dịch thuật đang chạy production bằng model mới, **chỉ khi model mới chứng minh hiệu suất tốt hơn**. Quy trình phải **tuần tự (sequential)**, mỗi bước được xem xét và phê duyệt trước khi sang bước sau. Hãy **sắp thứ tự** các bước sau:

- Phân tích kết quả và tạo báo cáo đánh giá toàn diện.
- Tiến hành A/B testing so sánh model mới với model production hiện tại.
- Tạo tập dữ liệu kiểm thử với kịch bản đa dạng và edge cases.
- Định nghĩa các metric đánh giá (độ liên quan, độ chính xác, độ trôi chảy).
- Triển khai automated quality gates (cổng chất lượng tự động) bằng AWS Step Functions.

<details>
<summary>📖 Xem phân tích & đáp án</summary>

### 🎯 Câu này test concept gì
Quy trình evaluation có hệ thống, **khoa học và tuần tự** — đặt thước đo trước, rồi mới đo. Thuộc Domain 5 (Testing, Validation & Troubleshooting).

### 🔍 Phân tích cốt lõi — bài toán "tổ chức kỳ thi tuyển nhân tài"
Đưa một AI mới thay AI cũ là việc cực rủi ro; bạn không thể "cảm thấy" nó giỏi hơn rồi dùng ngay. AWS bắt bạn theo nguyên tắc giống tổ chức một kỳ thi: **ra luật chấm thi → ra đề thi → cho thi thật → chấm qua cổng tự động → ra quyết định cuối**.

### ✅ Thứ tự đúng (và vì sao)
1. **Define evaluation metrics** — trước khi làm gì, phải biết **"thế nào là giỏi"** (độ chính xác bao nhiêu %, câu văn mượt ra sao). Đây là thước đo cho mọi bước sau.
2. **Create test dataset** — có luật chấm rồi, giờ chuẩn bị **"đề thi"**: bộ dữ liệu đủ câu khó, đủ edge cases, dùng chung để kiểm tra cả 2 model.
3. **Conduct A/B testing** — **"lên võ đài"**: cho cả model cũ (A) và mới (B) cùng giải đề ở bước 2, đo kết quả theo metric ở bước 1.
4. **Implement automated quality gates (Step Functions)** — **"giám khảo máy móc"**: Step Functions tự kiểm tra điểm model mới có vượt ngưỡng không, và **ép phải có người duyệt (approval)** mới đi tiếp.
5. **Analyze results & generate report** — sau khi vượt mọi cổng, **tổng hợp bằng chứng** thành báo cáo để lãnh đạo bấm nút "đồng ý thay model".

### 💡 Bỏ túi mẹo đi thi
Gặp dạng workflow/ordering về đánh giá model, nhớ thần chú: **Metrics → Dataset → Test → Gate/Threshold → Report**. Không bao giờ đưa dữ liệu vào test trước khi định nghĩa xong metric đánh giá.

</details>

---

## Câu 5 — Bắt buộc mọi lệnh gọi phải gắn Guardrail

> Bài toán **quản trị AI cấp doanh nghiệp**: làm sao **"ép"** tất cả lập trình viên khi gọi AI đều bắt buộc đi qua màng lọc an toàn (Guardrails), mà bạn không phải đi **canh chừng từng người**?

Một tập đoàn sản xuất triển khai chính sách quản trị AI: **mọi** lệnh gọi `InvokeModel` và `Converse` tới FM **bắt buộc** áp Bedrock Guardrails. Lập trình viên ở nhiều nhóm có thể "quên" gắn guardrail trong code. Cần giải pháp **hiệu quả vận hành nhất (MOST operationally efficient)**.

Giải pháp nào?

- **A.** Cấu hình IAM policy với cả `bedrock:GuardrailIdentifier` **và** `bedrock:PromptRouterArn`, yêu cầu xác thực prompt router trước khi truy cập.
- **B.** Viết một Lambda làm proxy xác thực và ép guardrail trước khi chuyển tiếp tới Bedrock; bắt mọi tương tác đi qua Lambda.
- **C.** Lưu guardrail ID trong Parameter Store; Lambda lấy ID mỗi lần trước khi gọi Bedrock.
- **D.** Cấu hình IAM policy với condition key **`bedrock:GuardrailIdentifier`**, áp cho mọi IAM role truy cập FM.

<details>
<summary>📖 Xem phân tích & đáp án</summary>

### 🎯 Câu này test concept gì
Dùng **IAM Policy + Condition Key** để ép tuân thủ ở **tầng hạ tầng** — thay vì tự viết code đi kiểm tra code của người khác. Thuộc Domain 3.

### 🔍 Phân tích cốt lõi — bài toán "ép buộc tuân thủ luật lệ"
Lập trình viên hoàn toàn có thể "quên" hoặc cố tình lờ việc chèn guardrail ID vào code. Nếu thiếu ID, AI trả lời mà **không có màng lọc** (chống chửi thề, chống lộ PII).

**Thuốc đặc trị — chốt chặn IAM:** cách quản trị đỉnh cao và nhàn nhất không phải đi viết code kiểm tra code, mà là dùng **IAM Policy với Condition Key**. Bạn đặt một luật ở cổng AWS: *"Bất kỳ ai gọi API Bedrock mà gói tin KHÔNG chứa `bedrock:GuardrailIdentifier` → tôi (AWS) đá văng ngay với lỗi Access Denied!"*. Giải pháp này **miễn phí, built-in, không tốn một giây bảo trì**.

### 🧩 Mổ xẻ từng đáp án
- **D — ĐÚNG (tiêu chuẩn vàng).** Dùng condition key `bedrock:GuardrailIdentifier` trong IAM policy là cách chính thống và hiệu quả nhất. Lệnh gọi nào không kèm guardrail ID, IAM **tự chặn ở cấp hạ tầng**. Không cần code tùy chỉnh nào → hiệu quả vận hành cao nhất.
- **A — SAI (vẽ rắn thêm chân).** Tuy có condition key đúng, nhưng lại bắt chèn thêm `bedrock:PromptRouterArn`. Prompt Router dùng để **điều hướng prompt**, hoàn toàn **không liên quan** đến cưỡng chế guardrail; thêm điều kiện thừa chỉ làm policy phức tạp vô nghĩa.
- **B — SAI (tự tạo nút thắt cổ chai).** Lambda proxy đứng giữa người dùng và Bedrock có thể **sập, quá tải, tăng latency** cho mọi truy vấn, và phải liên tục bảo trì code → vi phạm nghiêm trọng "operationally efficient".
- **C — SAI (chậm và tốn kém).** Như B đã trừ điểm vì dùng Lambda; tệ hơn, **mỗi lần gọi** Bedrock lại phải chạy đi hỏi Parameter Store lấy ID → độ trễ vô ích + tốn phí API hàng ngàn lần.

### ✅ Kết luận: **D.**

### 💡 Bỏ túi mẹo đi thi
1. **"Enforce Bedrock Guardrails" + "MOST operationally efficient"** → đáp án luôn là **IAM Policy + condition key `bedrock:GuardrailIdentifier`**.
2. **Tránh xa custom proxy:** khi AWS yêu cầu Access Control / Governance, luôn dùng **IAM**. Gạch mọi đáp án xúi dùng **Lambda làm proxy** đứng giữa để kiểm tra quyền — đó là thiết kế tồi, ngược triết lý serverless governance.

</details>

---

## Câu 6 — Dừng sinh văn bản tại một cụm từ

> Câu này về **tham số suy luận (inference parameters)** — cụ thể là **"cái phanh xe" của AI**: làm sao bắt AI **im lặng ngay** khi nó vừa viết đến một cụm từ định trước?

Một lập trình viên xây trợ lý ảo bằng Anthropic Claude trên Bedrock. Ứng dụng cần **ngừng sinh đầu ra ngay sau khi một cụm từ cụ thể được tạo ra** trong câu trả lời.

Giải pháp nào?

- **A.** Thêm câu "hãy dừng tại cụm từ này" vào user prompt.
- **B.** Dùng tham số **stop sequences (chuỗi dừng)** trong lệnh gọi suy luận để chỉ định cụm từ kích hoạt.
- **C.** Dùng tham số **top-k** để kiểm soát độ đa dạng token.
- **D.** Dùng tham số **temperature** để kiểm soát khả năng cụm từ xuất hiện.

<details>
<summary>📖 Xem phân tích & đáp án</summary>

### 🎯 Câu này test concept gì
Hiểu đúng chức năng từng inference parameter; nhận ra **stop sequences** là cơ chế dừng đáng tin cậy. Thuộc Domain 2.

### 🔍 Phân tích cốt lõi — bài toán "cái phanh xe"
Bạn muốn AI dừng ngay khi viết đến chữ "Kết luận:".
- **Cách ngây thơ:** dặn nó trong prompt — giống dặn tài xế dừng bằng miệng, đôi khi tài xế **"đi lố"**. LLM rất hay "lỡ lời", nhất là khi độ sáng tạo cao.
- **Thuốc đặc trị — cắt cầu dao tự động (stop sequences):** API Bedrock có tham số `stop_sequences = ["Kết luận:"]`. Đây là **cái phanh tự động của máy chủ**: ngay tích tắc server thấy AI định nhả chuỗi đó, nó **cắt kết nối ngay**, đảm bảo 100% AI không nói thêm một từ.

### 🧩 Mổ xẻ từng đáp án
- **B — ĐÚNG.** Stop sequences sinh ra chính xác cho mục đích này. Bạn truyền vào một mảng từ/ký tự (ví dụ `["\n\nHuman:", "Kết luận:"]`); khi model chuẩn bị sinh chuỗi đó, quá trình sinh token **ngắt ngay lập tức**. Đây là cơ chế kiểm soát kỹ thuật tin cậy 100%.
- **A — SAI (phụ thuộc tâm trạng AI).** Dặn qua prompt phụ thuộc khả năng tuân lệnh của model. LLM không phải máy if-else hoàn hảo; đang "bắt trớn" viết, nó có thể bỏ qua lời dặn và sinh tràn lan → thiếu tin cậy cho một tính năng cốt lõi.
- **C — SAI (hiểu sai Top-k).** Top-k giới hạn **số lượng từ vựng** model được cân nhắc cho token kế (ví dụ chọn 1 trong 50 từ xác suất cao nhất); ảnh hưởng vốn từ/đa dạng, **không có khả năng "dừng"**.
- **D — SAI (hiểu sai Temperature).** Temperature kiểm soát **độ ngẫu nhiên/sáng tạo** (cao thì bay bổng, thấp thì rập khuôn); hoàn toàn **không** ngắt hay dừng việc sinh văn bản.

### ✅ Kết luận: **B.**

### 💡 Bỏ túi mẹo đi thi (4 tham số kinh điển)
1. Muốn AI **NGỪNG** tại một ký tự/từ cụ thể → **stop sequences**.
2. Muốn AI bớt **ảo giác (hallucination)**, trả lời nghiêm túc, ổn định → **giảm temperature về 0**.
3. Muốn AI **sáng tạo**, làm thơ, viết truyện → **tăng temperature** gần 1.
4. Muốn kiểm soát **vốn từ vựng** (tránh từ kỳ quặc) → chỉnh **Top-P / Top-K**.

</details>

---

## Câu 7 — Tối ưu tài nguyên endpoint LLM (Chọn HAI)

> Bài toán **tối ưu hóa phần cứng và chi phí triển khai LLM**: giống như bạn thuê **xe buýt 50 chỗ nhưng chỉ chở 5 người** — vô cùng lãng phí! Làm sao nhồi đầy chiếc xe đó?

Một LLM fine-tuned được triển khai lên SageMaker AI endpoint với continuous batching (thư viện DJL). Mỗi EC2 có **8 GPU**. Khi lên production, cần quá nhiều instance → chi phí tăng. Log cho thấy: (1) độ dài chuỗi I/O thực tế **nhỏ hơn 10 lần** cấu hình, concurrency mỗi instance thấp; (2) trọng số + activations chỉ cần **4 GPU** là đủ.

Hai bước nào cải thiện mức sử dụng tài nguyên? (Chọn HAI.)

- **A.** Tăng số instance SageMaker và phân tán đều yêu cầu.
- **B.** **Giảm maximum sequence length** của model để có rolling batch size lớn hơn cho mỗi GPU.
- **C.** Bật speculative decoding để giảm latency mỗi yêu cầu.
- **D.** Dùng **tensor parallelism degree = 4** để triển khai **2 bản sao (replicas)** mô hình trên mỗi instance.
- **E.** Chia model trên cả 8 GPU bằng tensor parallelism degree = 8.

<details>
<summary>📖 Xem phân tích & đáp án</summary>

### 🎯 Câu này test concept gì
Tối ưu VRAM (bộ nhớ GPU) trong continuous batching, và bài toán chia tensor parallelism. Thuộc Domain 4 (Operational Efficiency).

### 🔍 Phân tích cốt lõi — 2 căn bệnh lãng phí
**Bệnh 1 — "đặt trước ghế nhưng không ai ngồi" (Sequence Length):** trong máy chủ LLM (DJL/vLLM), VRAM cho **KV Cache** bị khóa cứng theo cấu hình *Max Sequence Length*. Bạn cấu hình cho phép 10.000 từ → GPU trích lập dự phòng VRAM đủ 10.000 từ. Nhưng thực tế khách chỉ chat 1.000 từ → đống VRAM thừa **bỏ trống**, đáng lẽ dùng để phục vụ thêm khách cùng lúc (tăng batch size/concurrency).

**Bệnh 2 — "bếp lớn nướng bánh nhỏ" (Tensor Parallelism):** bạn thuê EC2 xịn 8 GPU, nhưng model chỉ cần 4 GPU. Để mặc định, model chạy trên 4 GPU và **4 GPU còn lại ngồi chơi**.

**Cách chữa:** (1) giảm giới hạn số từ cho sát thực tế → dư VRAM → tăng batch size (nhồi thêm khách). (2) tận dụng 8 GPU bằng cách **nhân bản** model: 8 GPU ÷ 4 GPU/model = **2 replicas** trên cùng 1 máy → nhân đôi công suất.

### 🧩 Mổ xẻ từng đáp án
- **B — ĐÚNG.** DJL quản bộ nhớ rất nghiêm. Khi người dùng thực tế xài ít chữ (nhỏ hơn 10 lần), giảm Max Sequence Length là hành động chính xác: VRAM được giải phóng khỏi việc "đặt cọc giữ chỗ" → dùng để xử lý nhiều luồng hơn cùng lúc (higher rolling batch size).
- **D — ĐÚNG.** Model vừa 4 GPU → đặt **TP = 4**; máy có 8 GPU → DJL tự khởi tạo **2 replicas** (4×2=8). Một máy nhưng sức xử lý bằng 2 máy gộp lại.
- **A — SAI (đốt tiền).** "Mua thêm instance" chính là nguyên nhân tăng chi phí mà đề muốn tránh; không giải quyết việc máy hiện tại đang lãng phí tài nguyên bên trong.
- **C — SAI (chữa sai bệnh).** Speculative decoding giúp **sinh từ nhanh hơn (giảm latency)**, nhưng không tiết kiệm VRAM hay tăng resource utilization — thậm chí **ngốn thêm VRAM** để chạy một draft model song song.
- **E — SAI (kéo dãn vô ích).** TP = 8 ép model vốn chỉ cần 4 GPU rải mảnh lên cả 8 GPU; máy vẫn chỉ **1 replica**, hiệu suất không tăng mà còn chậm hơn do 8 GPU phải liên tục giao tiếp (communication overhead) → lãng phí cơ hội tạo replica thứ 2.

### ✅ Kết luận: **B + D.**

### 💡 Bỏ túi mẹo đi thi
1. **Tensor Parallelism:** luôn làm phép chia — EC2 có **X** GPU, model fit vào **Y** GPU → cấu hình tốt nhất là **TP = Y**, số replica = **X / Y**. Tuyệt đối không dàn TP = X nếu không cần.
2. **Continuous Batching / VRAM / Sequence Length** là quan hệ **"cái bập bênh":** giảm Max Sequence Length → dư VRAM → tăng được Max Batch Size → phục vụ nhiều khách cùng lúc hơn.

</details>

---

## Câu 8 — Stream gợi ý real-time lên web editor

> Trải nghiệm quen thuộc: làm sao để AI **"gõ từng chữ" (streaming)** lên màn hình ngay lập tức, thay vì bắt người dùng nhìn biểu tượng xoay vòng chờ cả chục giây?

Một hãng luật xây trình soạn thảo web: khi luật sư bấm "phân tích", hệ thống phải **lập tức bắt đầu** stream gợi ý chỉnh sửa điều khoản (hiệu ứng gõ từng chữ) qua giao diện. Dùng Bedrock FM. Yêu cầu **ít công sức vận hành nhất**.

Kiến trúc nào?

- **A.** SQS nhận bài → Step Functions xử lý → Lambda → DynamoDB → API Gateway WebSocket stream về.
- **B.** **API Gateway WebSocket** liên kết **Lambda**; Lambda đọc metadata để định tuyến model, dùng **Bedrock Prompt Management** áp luật style, và **Bedrock streaming API** trả real-time.
- **C.** API Gateway **REST API** + Lambda function URLs, stream bằng chunked transfer encoding.
- **D.** Application Load Balancer + ECS (custom container) chạy logic định tuyến + Bedrock streaming, WebSocket về web.

<details>
<summary>📖 Xem phân tích & đáp án</summary>

### 🎯 Câu này test concept gì
Kiến trúc **response streaming real-time lên web UI**, toàn serverless. Thuộc Domain 2.

### 🔍 Phân tích cốt lõi — bài toán "AI gõ chữ trực tiếp"
Nếu dùng HTTP thông thường (gửi yêu cầu → chờ xử lý xong toàn bộ → trả về một cục), trải nghiệm rất tệ. Để có hiệu ứng "gõ từng chữ" như ChatGPT, web phải duy trì một **"đường ống kết nối mở liên tục"** với máy chủ: vừa nghĩ ra chữ nào, server đẩy ngay chữ đó về màn hình.

**Thuốc đặc trị — WebSocket + Bedrock Streaming API:**
1. **WebSocket (đường ống 2 chiều):** API Gateway hỗ trợ WebSocket API, cho trình duyệt và server kết nối liên tục, không ngắt quãng.
2. **Bedrock Streaming API:** thay vì `InvokeModel` thường, dùng `InvokeModelWithResponseStream` — AI nghĩ ra chữ nào ném ngay vào đường ống WebSocket bay thẳng về màn hình luật sư. Nhanh, mượt, không độ trễ.

### 🧩 Mổ xẻ từng đáp án
- **B — ĐÚNG (tiêu chuẩn vàng).** API Gateway WebSocket + Lambda + Bedrock Streaming API là Best Practice cho chatbot/trợ lý AI real-time. Mọi dịch vụ đều serverless → "ít công sức vận hành nhất". Lambda đóng vai cảnh sát giao thông siêu nhanh đọc metadata rồi chuyển hướng prompt phù hợp; Prompt Management quản style.
- **A — SAI (hàng đợi bóp nghẹt real-time).** Nhét **SQS** ở đầu nguồn là sai lầm chết người: SQS là kiến trúc **bất đồng bộ (async)**, phải "đợi" (poll) lấy tin nhắn → tạo độ trễ, phá hỏng "immediate feedback" mà luật sư cần khi bấm nút.
- **C — SAI (sai giao thức).** REST API thiết kế cho kết nối **ngắn hạn, đồng bộ** (gửi 1 lần - nhận 1 lần rồi đóng); không giữ kết nối mở lâu để stream mượt như WebSocket. Tác vụ AI mất hàng chục giây dễ bị **timeout**.
- **D — SAI (tự mua dây buộc mình).** Dựng cụm ECS, tự viết Dockerfile, tự quản container chỉ để chạy một logic định tuyến nhỏ là **"dùng dao mổ trâu giết gà"** — overhead cao, ngược yêu cầu.

### ✅ Kết luận: **B.**

### 💡 Bỏ túi mẹo đi thi
1. **"Real-time feedback / typewriter effect / streaming to web UI"** → cặp bài trùng **API Gateway WebSocket + Bedrock Streaming API**.
2. **Loại SQS khỏi real-time UI:** SQS tuyệt cho background processing, nhưng tuyệt đối **không** dùng khi người dùng đang bấm nút chờ kết quả ngay trên màn hình.
3. **Serverless > Containers:** có "LEAST operational overhead" → ưu tiên Lambda/API Gateway, mạnh dạn gạch các đáp án đòi tự dựng EC2/ECS.

</details>

---

## Câu 9 — Quản trị prompt + lưu log tuân thủ dài hạn (Chọn HAI)

> Hai yêu cầu sống còn của ngành tài chính: **quản lý quy trình duyệt của con người (governance)** và **lưu nhật ký chống xóa sửa để phục vụ thanh tra (compliance/audit)**.

Một công ty bảo hiểm dùng Bedrock cho trợ lý CSKH nhiều đơn vị kinh doanh. Cần: (1) prompt templates được quản trị qua **approval workflow (quy trình phê duyệt)**; (2) **ghi log toàn bộ lệnh gọi model** và **giữ 7 năm** để tuân thủ pháp luật. Yêu cầu **công sức vận hành tối thiểu**.

Hai bước nào? (Chọn HAI.)

- **A.** Dùng **Amazon Bedrock Prompt Management** với approval workflow nhiều giai đoạn.
- **B.** Lưu prompt trong DynamoDB, tự viết IAM + quyền item-level để kiểm soát phê duyệt.
- **C.** EventBridge bắt sự kiện invocation → CloudWatch Logs → export S3, bật S3 Object Lock 7 năm.
- **D.** Bật CloudTrail data events cho mọi API Bedrock, đưa vào CloudTrail Lake giữ 7 năm.
- **E.** Bật **Bedrock model invocation logging** đích S3, bật **S3 Object Lock (compliance mode) 7 năm**, tạo prefix riêng từng đơn vị.

<details>
<summary>📖 Xem phân tích & đáp án</summary>

### 🎯 Câu này test concept gì
Ghép 2 mảnh: (1) quản trị prompt bằng tính năng managed; (2) lưu **nội dung** chat bất biến 7 năm. Thuộc Domain 3.

### 🔍 Phân tích cốt lõi — 2 vế rõ ràng
**Vế 1 — quản lý & phê duyệt prompt:** trong công ty lớn, lập trình viên không thể tự sửa prompt rồi đưa cho khách dùng ngay. Phải có quy trình: viết nháp → gửi sếp duyệt → lên production. AWS có sẵn tính năng "chính chủ" cho việc này.

**Vế 2 — ghi log phục vụ thanh tra tài chính:** thanh tra không chỉ muốn biết "ai gọi AI lúc mấy giờ", họ muốn biết **chính xác khách chat gì và AI trả lời gì (full payload)**. Hơn nữa, file log phải bị **"khóa chết" 7 năm**, không ai (kể cả Giám đốc IT, kể cả tài khoản Root) xóa/sửa được.

### 🧩 Mổ xẻ từng đáp án
- **A — ĐÚNG (vế 1).** Amazon Bedrock Prompt Management là tính năng built-in: cung cấp sẵn giao diện và luồng tạo → gắn biến (parameterize) → đánh version → **approval workflow**. Dùng đồ có sẵn giảm tối đa công sức so với tự xây.
- **E — ĐÚNG (vế 2).** **Model invocation logging** ghi lại **trọn vẹn** prompt + response rồi ném thẳng vào S3. **S3 Object Lock chế độ Compliance** là mức bảo vệ cao nhất (WORM — Write Once Read Many): đã khóa thì **không quyền lực nào trên AWS** xóa/ghi đè trước hạn 7 năm — chuẩn bắt buộc của quy định tài chính (SEC/FINRA).
- **B — SAI (chế lại bánh xe).** Lưu prompt vào DynamoDB thì được, nhưng phải **tự code web quản lý + tự viết logic approval workflow** → overhead khổng lồ và không cần thiết khi đã có đáp án A.
- **C — SAI (đường vòng dễ thất thoát).** Chuỗi EventBridge → CloudWatch → export S3 rất cồng kềnh, nhiều điểm đứt gãy, và **không đảm bảo chụp được toàn bộ payload** của các câu chat dài → vi phạm "minimal overhead".
- **D — SAI (bẫy kinh điển về CloudTrail).** CloudTrail như camera an ninh ngoài cửa: chỉ ghi **metadata** (User John gọi `InvokeModel` lúc 10h00). Nó **không bao giờ lưu nội dung prompt/response**. Với thanh tra tài chính, không biết nhân viên hỏi AI gì là **vô giá trị**.

### ✅ Kết luận: **A + E.**

### 💡 Bỏ túi mẹo đi thi
1. **"Prompt versions / approval workflows"** → chọn ngay **Amazon Bedrock Prompt Management**.
2. **"Regulatory compliance / cannot be deleted / N-year retention"** → vũ khí tối thượng là **Amazon S3 Object Lock (Compliance mode)**.
3. **Phân biệt rạch ròi log AI:** cần biết **ai gọi API** → CloudTrail; cần biết **nội dung chat (prompt/response payload)** → **Bedrock Model Invocation Logging**. Đừng bao giờ chọn CloudTrail khi đề yêu cầu kiểm duyệt **nội dung**.

</details>

---

## Câu 10 — Triển khai agent Python lên AgentCore Runtime (Chọn HAI)

> Khi bạn đã có đoạn code Python xịn để làm AI Agent, làm sao biến nó thành một service chuẩn chỉnh trên Bedrock mà **không phải làm "thợ xây"** (tự setup server, tự đóng gói Docker, tự lo bảo mật)?

Một công ty fintech phải triển khai **code Python agent có sẵn** lên **Amazon Bedrock AgentCore Runtime**, muốn **giảm tối đa việc quản lý hạ tầng**. Agent phải xử lý cả tra cứu nhanh (dưới 1 giây) lẫn báo cáo dài (stream vài phút). Giải pháp phải **tự động** lo HTTP server, endpoint routing, health monitoring.

Hai cách nào với **overhead tối thiểu**? (Chọn HAI.)

- **A.** Tự dựng FastAPI server cấu hình endpoint `/invocations`, `/ping` và tự điều phối container.
- **B.** Dùng **AgentCore SDK** với decorator **`@app.entrypoint`** để tự động lo server + endpoint.
- **C.** Triển khai trên ECS Fargate bằng custom container chạy AgentCore SDK.
- **D.** Triển khai trên SageMaker AI real-time endpoint bằng custom inference container.
- **E.** Dùng **AgentCore starter toolkit** để tự động đóng gói, container hóa, và deploy.

<details>
<summary>📖 Xem phân tích & đáp án</summary>

### 🎯 Câu này test concept gì
Bộ công cụ AgentCore giúp đưa code lên Runtime với ít hạ tầng nhất. Thuộc Domain 2.

### 🔍 Phân tích cốt lõi — bệnh "viết code thì ít mà dựng server thì nhiều"
Để Bedrock nói chuyện được với file Python của bạn, nó phải thành một **HTTP server** mở cổng, có `/ping` báo cáo sức khỏe và `/invocations` nhận yêu cầu. Bắt lập trình viên AI tự viết mấy server này, tự đóng gói Docker, tự push lên ECR thì **kiệt sức**.

**Thuốc đặc trị — AgentCore SDK & Starter Toolkit** cứu lập trình viên ở 2 cấp độ:
1. **Cấp độ Code (`@app.entrypoint`):** thay vì viết cả một FastAPI dài dòng, chỉ cần gõ **một dòng** `@app.entrypoint` đặt trên hàm Python. AWS tự bọc hàm đó thành server chuẩn, hỗ trợ cả phản hồi nhanh lẫn streaming dài.
2. **Cấp độ Triển khai (starter toolkit):** thay vì tự `docker build`/`docker push` cồng kềnh, toolkit **tự đóng gói code, tự container hóa, tự deploy** lên Bedrock.

### 🧩 Mổ xẻ từng đáp án
- **B — ĐÚNG (cấp độ code).** Decorator `@app.entrypoint` là "vũ khí bí mật" thỏa mãn yêu cầu "tự động quản lý HTTP server", loại bỏ hoàn toàn việc tự code web framework. Hỗ trợ mượt cả JSON nhanh (dưới 1 giây) lẫn streaming (vài phút).
- **E — ĐÚNG (cấp độ hạ tầng).** Starter toolkit lo trọn phần hạ tầng tẻ nhạt (Dockerfile, ECR, AgentRuntime) → con đường ngắn nhất, ít tốn sức nhất (minimal overhead).
- **A — SAI (chế lại bánh xe).** Tự viết FastAPI = tự lo cổng, tự viết logic `/ping` và `/invocations`, tự xử lý streaming → overhead tăng vọt.
- **C, D — SAI (sai nền tảng).** Mang code chạy trên **ECS Fargate** (C) hay **SageMaker custom container** (D) đều bắt tự quản vòng đời Docker, định nghĩa Task, cấu hình Load Balancer/Auto Scaling. Trong khi đề đã chỉ định đích đến là **AgentCore Runtime** — đi vòng vừa sai yêu cầu vừa tốn sức.

### ✅ Kết luận: **B + E.**

### 💡 Bỏ túi mẹo đi thi
1. Nhắc đến **AgentCore Runtime** + "Minimal overhead / No manual server config" → săn 2 từ khóa: **`@app.entrypoint`** (cho code) và **AgentCore starter toolkit** (cho triển khai).
2. Đề hỏi cách **nhanh nhất, nhàn nhất** → mạnh dạn gạch các đáp án bắt tự viết web framework (FastAPI/Flask) hoặc tự build custom container.

</details>

---

## Câu 11 — Truy xuất nguồn gốc nội dung sinh ra (Chọn HAI)

> Bài toán **truy xuất nguồn gốc (data lineage)** trong GenAI: khi AI sinh ra một nội dung, làm sao người kiểm duyệt biết nó lấy kiến thức từ **nguồn nào** để xác minh tính chính xác?

Một nhà xuất bản giáo dục xây hệ thống sinh câu hỏi luyện tập trên Bedrock, dùng hỗn hợp dữ liệu **tuyển chọn (curated)** và **cào web (scraped)**. Người kiểm duyệt phải xác minh **độ tin cậy** bằng cách biết nội dung được lấy từ **nguồn nào (source lineage)**. Yêu cầu **ít công sức vận hành nhất**.

Hai bước nào? (Chọn HAI.)

- **A.** Bật Bedrock invocation logging rồi tự viết hệ thống tương quan log với nguồn dữ liệu.
- **B.** **Gắn metadata nguồn dữ liệu vào output** của FM (tag).
- **C.** **Đăng ký** các tập dữ liệu (curated + scraped) vào **AWS Glue Data Catalog**.
- **D.** Dùng SageMaker Clarify để giải thích dự đoán của model.
- **E.** Dùng CloudTrail để log hành động phê duyệt của người kiểm duyệt.

<details>
<summary>📖 Xem phân tích & đáp án</summary>

### 🎯 Câu này test concept gì
Source lineage = dán **"tem nguồn"** lên output + có một **"sổ đăng ký nguồn"** trung tâm. Thuộc Domain 1 & 3.

### 🔍 Phân tích cốt lõi — bài toán "cái tem nguồn gốc"
Hình dung hệ thống AI là nhà máy làm bánh, nguyên liệu lấy từ 2 nguồn: **nông trại chuẩn VietGAP** (curated) và **chợ trời** (scraped). Người kiểm duyệt (thanh tra an toàn thực phẩm) nhìn cái bánh thành phẩm (câu hỏi AI sinh ra) và hỏi: *"Bánh này dùng bột từ đâu?"*. Để trả lời mà không phải lật tung cả nhà máy, cần 2 việc:
1. **Dán tem lên cái bánh:** ngay khi bánh ra lò, dán một **metadata tag** ghi mã lô hàng.
2. **Cuốn sổ đăng ký nguồn:** liệt kê mã 1 = Nông trại A, mã 2 = Chợ B. Thanh tra nhìn mã trên tem, mở sổ đối chiếu là xong.

### 🧩 Mổ xẻ từng đáp án
- **B — ĐÚNG (dán tem).** Lập trình để mỗi khi AI sinh output, hệ thống tự nhúng **metadata của tài liệu gốc** vào. Người kiểm duyệt thấy ngay tag `Source: Wikipedia_Biology_p42`. Cách trực tiếp, tự động, cực nhẹ.
- **C — ĐÚNG (sổ đăng ký).** AWS Glue Data Catalog đóng vai "cuốn sổ đăng ký" trung tâm: lập chỉ mục và quản lý mọi nguồn (curated + scraped). Thấy tag ở bước B, kiểm duyệt vào Data Catalog tra cứu nguồn có uy tín không. Là **managed service** → đáp ứng "least overhead".
- **A — SAI (thủ công, phức tạp).** Invocation logging chỉ ghi "có người gửi prompt X, nhận output Y"; **không biết AI lấy kiến thức từ file PDF nào**. Muốn "tương quan" log với kho tài liệu phải tự viết hệ thống đối chiếu cực phức tạp, tốn nhiều công.
- **D — SAI (dùng sai công cụ).** SageMaker Clarify chuyên phát hiện **bias** và **explainability** của ML cổ điển (vì sao AI duyệt khoản vay này). Nó **không** truy vết văn bản nguồn (source lineage) cho GenAI.
- **E — SAI (lạc đề).** CloudTrail chỉ theo dõi **hành động con người** trên AWS (API calls): "anh A bấm nút Phê duyệt lúc 3 giờ chiều". Hoàn toàn mù tịt về nguồn dữ liệu của câu hỏi.

### ✅ Kết luận: **B + C.**

### 💡 Bỏ túi mẹo đi thi
1. **"Source lineage / data origin / traceability"** → cặp đôi **Metadata Tagging** (gắn vào output) + **AWS Glue Data Catalog** (kho metadata trung tâm).
2. **Đừng nhầm Explainability với Lineage:** Clarify giải thích **CÁCH** model suy nghĩ; Data Catalog giải thích **NƠI** model lấy dữ liệu đầu vào.

</details>

---

## Câu 12 — RAG "hỏng âm thầm" sau khi cập nhật code

> Vào vai **"thám tử AI"** gỡ lỗi RAG: hệ thống không hề báo lỗi đỏ, mọi thứ trông hoàn hảo, nhưng kết quả trả ra **sai bét**!

Một công ty tài chính chạy RAG: Bedrock làm embedding model, OpenSearch làm vector store, Lambda lo logic nhúng + tìm kiếm. **Sau một đợt cập nhật code Lambda**, ứng dụng bắt đầu trả "không tìm thấy thông tin liên quan" ngay cả với câu trước đây trả lời tốt. CloudWatch **không có lỗi**, X-Ray xác nhận **gọi FM thành công**, OpenSearch khỏe mạnh, latency bình thường.

Nguyên nhân?

- **A.** Vector tài liệu trong OpenSearch bị xóa khi cập nhật và chưa re-index.
- **B.** IAM role của Lambda thiếu quyền `bedrock:InvokeModel`.
- **C.** Tham số temperature của FM bị tăng.
- **D.** Code Lambda mới dùng **một phiên bản embedding model khác**.

<details>
<summary>📖 Xem phân tích & đáp án</summary>

### 🎯 Câu này test concept gì
**Embedding drift / vector space mismatch** — và kỹ năng đọc "dấu vết hiện trường" để loại trừ. Thuộc Domain 5.

### 🔍 Phân tích cốt lõi — bài toán "ông nói gà, bà nói vịt"
Hình dung ban đầu toàn bộ tài liệu được "dịch sang **tiếng Nhật**" (embedding model V1) rồi lưu vào thư viện OpenSearch. Hôm nay lập trình viên cập nhật code Lambda, vô tình đổi sang model "**tiếng Pháp**" (V2). Khi người dùng hỏi, Lambda dịch câu hỏi sang "tiếng Pháp" rồi mang vào thư viện toàn "tiếng Nhật" để tìm tương đồng.

Về kỹ thuật, **hệ thống không hề lỗi**: Lambda chạy tốt, OpenSearch khỏe, không dòng log lỗi nào. Nhưng tiếng Pháp và tiếng Nhật **lệch hệ tọa độ toán học** → OpenSearch trả về 0 kết quả → "không tìm thấy thông tin liên quan". Đây là hiện tượng **Embedding Drift**.

### 🧩 Mổ xẻ từng đáp án
- **D — ĐÚNG.** Đổi version embedding (ví dụ `titan-embed-text-v1` → `v2`) tạo một **không gian vector hoàn toàn khác**. Code mới biến query thành hệ tọa độ mới không so khớp được với tài liệu cũ. Lỗi diễn ra **"âm thầm"** nên không có log lỗi nào (khớp dữ kiện CloudWatch sạch).
- **A — SAI.** Cập nhật **code Lambda** không thể tự động chọc vào OpenSearch xóa sạch dữ liệu; hơn nữa nếu DB trống thường có cảnh báo truy vấn rỗng. Trọng tâm bài toán nằm ở luồng cập nhật code Lambda.
- **B — SAI (phủ định dữ kiện).** Đề ghi rõ **"X-Ray xác nhận gọi FM thành công"**. Nếu thiếu quyền IAM, X-Ray/CloudWatch sẽ **ngập lỗi `AccessDeniedException`** và app crash ngay, chứ không nhẹ nhàng trả "không tìm thấy".
- **C — SAI (hiểu sai Temperature).** Temperature chỉ ảnh hưởng **bước 2 (Generation - sinh văn bản)**. Nhưng câu "không tìm thấy thông tin" chứng tỏ thất bại ngay từ **bước 1 (Retrieval - tìm kiếm)**; LLM không tìm thấy tài liệu gốc nên bó tay. Temperature không liên quan khả năng tìm kiếm của OpenSearch.

### ✅ Kết luận: **D.**

### 💡 Bỏ túi mẹo đi thi
1. **Silent failure trong RAG:** đề mô tả *hệ thống đang chạy ngon → cập nhật code → mọi thứ vẫn "xanh" (không lỗi, latency tốt) → nhưng tìm kiếm ra rỗng/sai bét* → gần như chắc chắn là **đổi phiên bản embedding model**.
2. **Quy tắc vàng:** không gian vector của model A **không bao giờ** tương thích model B (kể cả v1 và v2 cùng hãng). Đổi embedding model thì **BẮT BUỘC re-index** toàn bộ database.

</details>

---

## Câu 13 — Giám sát quá trình nạp tài liệu vào Knowledge Base

> Việc của kỹ sư vận hành: khi bạn ném 1.000 file vào Bedrock để nó đọc, làm sao biết **file nào nạp thành công, file nào lỗi** (do mã hóa mật khẩu, do quá lớn, do PDF hỏng)?

Một công ty chạy ứng dụng hỏi đáp dùng Bedrock Knowledge Base nạp tài liệu từ nhiều bucket S3. Cần **giám sát quá trình ingestion (nạp dữ liệu)** để phát hiện và khắc phục **tài liệu nào xử lý lỗi**.

Giải pháp nào?

- **A.** Cấu hình **knowledge base logging** đích **CloudWatch Logs**, dùng **CloudWatch Logs Insights** truy vấn tài liệu xử lý lỗi.
- **B.** Bật CloudWatch Application Signals để tự phát hiện và cảnh báo vấn đề hiệu suất KB.
- **C.** Bật CloudTrail theo dõi mọi API liên quan KB và ingestion.
- **D.** Bật Bedrock model invocation logging để thu metric chi tiết về xử lý tài liệu và embedding.

<details>
<summary>📖 Xem phân tích & đáp án</summary>

### 🎯 Câu này test concept gì
Phân biệt 3 loại log Bedrock; chọn **Knowledge Base logging** cho việc theo dõi ingestion ở cấp độ từng tài liệu. Thuộc Domain 5.

### 🔍 Phân tích cốt lõi — bài toán "kiểm hàng nhập kho"
Khi KB nạp dữ liệu từ S3, nó chạy một **"Ingestion Job"** gồm: mở file → đọc chữ → cắt đoạn (chunking) → biến thành vector (embedding) → lưu vào DB (indexing). Trong quá trình này, lỗi xảy ra **ở cấp độ từng file** (PDF hỏng, file mã hóa mật khẩu, ảnh không có text). Bạn cần một bản báo cáo chi tiết: *File A - Success, File B - Failed (lý do), File C - Ignored*.

**Thuốc đặc trị — Knowledge Base logging:** AWS có tính năng ghi log **chuyên biệt cho KB**, ghi trạng thái sinh tử của từng tài liệu (`SUCCESS`, `RESOURCE_IGNORED`, `EMBEDDING_FAILED`, `INDEXING_FAILED`) và gửi thẳng vào CloudWatch Logs. Có hàng vạn dòng log, bạn dùng **CloudWatch Logs Insights** (như công cụ SQL cho log) gõ lệnh lọc ngay: *"cho tôi tất cả file `INDEXING_FAILED`"*.

### 🧩 Mổ xẻ từng đáp án
- **A — ĐÚNG.** Knowledge base logging sinh ra **chính xác** cho mục đích này: gửi log vào CloudWatch Logs để theo dõi trạng thái **từng file**; kết hợp Logs Insights để truy vấn và phân tích nguyên nhân gốc rễ cực nhanh.
- **B — SAI (sai chức năng).** CloudWatch Application Signals là công cụ giám sát hiệu năng ứng dụng (APM) — theo dõi microservices, API nào nghẽn (trên EKS/API Gateway). Nó **không** đọc log nạp tài liệu của KB.
- **C — SAI (bẫy quen thuộc).** CloudTrail chỉ ghi **hành động API** (audit): "anh A bấm `StartIngestionJob` lúc 10h". Nhưng nó **không biết** trong bucket có bao nhiêu file và `baocao.pdf` thành công hay lỗi — không có độ sâu **document-level**.
- **D — SAI (sai loại log).** Model invocation logging chỉ ghi nội dung lúc người dùng **chat (inference)** — prompt & response. Hoàn toàn không liên quan đến giai đoạn **ingestion** ngầm của KB.

### ✅ Kết luận: **A.**

### 💡 Bỏ túi mẹo đi thi (3 loại log Bedrock — đừng bị đánh lừa)
1. Cần biết **ai gọi API, lúc mấy giờ** → **AWS CloudTrail**.
2. Cần biết **khách chat gì, AI trả lời ra sao (input/output payload)** → **Bedrock Model Invocation Logging**.
3. Cần biết **file nào lỗi khi đẩy vào RAG (ingestion status)** → **Knowledge Base Logging + CloudWatch Logs Insights**.

</details>

---

## Câu 14 — Tối ưu năng suất với Amazon Q Developer (Chọn HAI)

> Làm sao tận dụng **"trợ lý lập trình AI"** triệt để nhất — vừa code nhanh hơn, vừa tự động hóa kiểm thử — mà **không làm chậm** quy trình bằng các chốt chặn thủ công?

Một nhóm liên chức năng xây ứng dụng GenAI, muốn **tối ưu năng suất lập trình viên**, thực thi pattern tích hợp nhất quán, **tự động hóa tinh chỉnh hiệu suất** và **tăng tốc kiểm thử AI**. Nhóm dùng Amazon Q Developer.

Hai bước nào? (Chọn HAI.)

- **A.** Dùng Q Developer phân tích bảo mật nhưng **bắt mọi thay đổi code phải được đội bảo mật duyệt thủ công** trước khi tích hợp.
- **B.** Dùng Q Developer **phân tích hồi tố (retrospective)** và viết tài liệu các pattern sau khi code đã viết xong.
- **C.** Cấu hình Q Developer **tự động sinh/refactor code tích hợp, gợi ý tối ưu hiệu suất** cho thành phần AI, áp dụng trên codebase module.
- **D.** Dùng Q Developer vá lỗi trong merge request, nhưng để dành phần lớn refactor/tối ưu cho **chu kỳ review thủ công định kỳ**.
- **E.** Tích hợp tính năng **tự sinh unit/integration test** của Q Developer vào **CI/CD pipeline**.

<details>
<summary>📖 Xem phân tích & đáp án</summary>

### 🎯 Câu này test concept gì
Dùng Q Developer một cách **chủ động (proactive) và tự động (automated)**, thay vì thụ động hoặc chen ngang thủ công. Thuộc Domain 2 & 4.

### 🔍 Phân tích cốt lõi — "tối đa hóa sức mạnh trợ lý AI"
Amazon Q Developer không phải chatbot hỏi đáp lặt vặt — nó là **AI coding companion** sống ngay trong IDE và trong cả quy trình DevOps (CI/CD). Đề yêu cầu 3 việc khó: (1) tối ưu năng suất (code nhanh, chuẩn hơn); (2) tự động hóa tinh chỉnh hiệu suất; (3) tăng tốc kiểm thử. Muốn đạt, phải dùng Q Developer **chủ động + tự động**, **không** tạo bottleneck thủ công.

### 🧩 Mổ xẻ từng đáp án
- **C — ĐÚNG (năng suất & hiệu suất).** Mô tả việc dùng Q Developer toàn diện và chủ động ngay khi viết code: tự sinh code, refactor cho gọn, và đặc biệt **gợi ý tối ưu hiệu suất** — khớp chính xác "tự động hóa tinh chỉnh hiệu suất".
- **E — ĐÚNG (kiểm thử).** Gắn thẳng tính năng **tự sinh test** của Q Developer vào CI/CD (CodePipeline/GitHub Actions): có code mới đẩy lên là AI tự viết test và chạy → hoàn toàn tự động, khớp "accelerate AI testing".
- **A — SAI (nút thắt cổ chai con người).** Quét bảo mật thì tốt, nhưng vế "**mọi thay đổi phải được duyệt thủ công**" tạo bottleneck khổng lồ, phá hỏng mục tiêu "accelerate workflows".
- **B — SAI (muộn màng, thụ động).** "Retrospective" = đợi code xong xuôi rồi mới lôi Q Developer ra đọc lại + viết tài liệu — cách "chữa cháy" thụ động, không giúp viết code nhanh hơn ngay từ đầu.
- **D — SAI (đánh mất giá trị AI).** Bảo AI chỉ bắt lỗi lặt vặt, còn refactor/tối ưu để con người làm thủ công định kỳ — đi ngược mục đích dùng AI để giải phóng sức lao động. Có AI tối ưu (như C) mà không xài, lại cày tay?

### ✅ Kết luận: **C + E.**

### 💡 Bỏ túi mẹo đi thi
1. **Từ khóa ăn điểm:** với Q Developer + mục tiêu "productivity / accelerate" → chọn các đáp án nói về **Automated, Generate, Refactor, Test generation trong CI/CD**.
2. **Từ khóa chết chóc:** gạch ngay các đáp án chứa **Manually approved** (duyệt thủ công), **Manual review cycles** (review thủ công), hoặc **Retrospectively** (hồi tố/làm sau). AI sinh ra để chạy **tự động và real-time**.

</details>

---

## Câu 15 — Chọn loại SageMaker inference cho sinh ảnh

> Bài toán **chọn đúng "loại xe chở hàng"**: SageMaker có 4 kiểu inference, mỗi kiểu một đặc thù. Mô hình sinh ảnh siêu nặng cần loại nào để không bị "quá tải" hay "hết giờ chờ"?

Một lập trình viên triển khai model sinh ảnh từ text (thử qua JumpStart). Cần triển khai cho người dùng tạo ảnh **theo yêu cầu (on demand)**, với 3 điều kiện: **dùng GPU**, **payload tới 40 MB**, **phản hồi trong vòng 10 phút**.

Chiến lược nào?

- **A.** **SageMaker Asynchronous Inference** dùng instance accelerated computing (GPU); Lambda gọi endpoint on-demand.
- **B.** SageMaker Serverless Inference dùng instance đa dụng; Lambda gọi endpoint.
- **C.** SageMaker Real-Time Inference dùng instance GPU; Lambda gọi endpoint.
- **D.** SageMaker batch transform job dùng instance GPU; Lambda khởi chạy job.

<details>
<summary>📖 Xem phân tích & đáp án</summary>

### 🎯 Câu này test concept gì
Chọn loại SageMaker inference theo đặc thù **payload lớn + xử lý dài + cần GPU + on-demand**. Thuộc Domain 2 & 4.

### 🔍 Phân tích cốt lõi — đối chiếu 3 yêu cầu khắt khe
- **Cần GPU** → bắt buộc instance **Accelerated Computing**.
- **Payload 40 MB** → rất nặng (lệnh gọi API thường chỉ vài MB).
- **Chờ 10 phút** → rất lâu (API thường timeout sau ~60 giây).
- **On-demand** → khách bấm nút là chạy, không phải gom cuối ngày chạy một mẻ.

**Thuốc đặc trị — Asynchronous Inference:** loại này sinh ra cho model siêu nặng (vẽ ảnh độ phân giải cao). Nó cho phép **payload khổng lồ (tới ~1 GB)** — dư sức cân 40 MB — và cho model ngồi "suy nghĩ" **rất lâu** — thừa cho 10 phút. Lại có **hàng đợi nội bộ** để quản lý các yêu cầu on-demand mượt mà.

### 🧩 Mổ xẻ từng đáp án
- **A — ĐÚNG.** Asynchronous Inference hỗ trợ instance GPU (accelerated computing), payload lớn và thời gian xử lý dài, có internal queue cho on-demand → khớp cả 40 MB lẫn 10 phút.
- **B — SAI (không có GPU).** Serverless Inference rất tiện vì không phải quản máy chủ, nhưng **điểm yếu chí mạng là KHÔNG hỗ trợ GPU**. Sinh ảnh không GPU thì không chạy nổi hoặc chậm như rùa.
- **C — SAI (quá tải & timeout).** Real-Time thiết kế cho tác vụ **siêu nhanh (mili-giây)** nên bị giới hạn khắt khe: payload tối đa **~6 MB** (đề cần 40 MB) và timeout **~60 giây** (đề cần 10 phút) → báo lỗi ngay.
- **D — SAI (offline, không hợp).** Batch transform là **xử lý hàng loạt ngoại tuyến** — gom file cuối ngày chạy một mẻ. Không thiết kế cho ứng dụng trực tiếp nơi khách bấm nút muốn ảnh ngay (on-demand).

### ✅ Kết luận: **A.**

### 💡 Bỏ túi mẹo đi thi (cheat sheet SageMaker inference)
- **Real-time:** phản hồi mili-giây, payload nhỏ (~6 MB), timeout ngắn (~60s) → chatbot chat trực tiếp.
- **Asynchronous:** payload khổng lồ (~1 GB), thời gian dài, có queue tích hợp → **sinh ảnh (computer vision), xử lý video**.
- **Serverless:** không muốn quản server, traffic trồi sụt (spiky), nhưng **không hỗ trợ GPU**.
- **Batch Transform:** xử lý offline, phân tích cả cục data lớn ban đêm, không cho app tương tác.

</details>

---

## Câu 16 — Vector search khối lượng lớn, tần suất thấp, rẻ nhất

> Bài toán tối ưu chi phí kiểu **"nuôi quân 3 năm, dùng 1 giờ"**: lưu một lượng dữ liệu khổng lồ và **thỉnh thoảng mới tìm kiếm**, mà không "cháy túi" vì phí duy trì máy chủ.

Một ứng dụng quan sát Trái Đất cần **similarity search trên 80 triệu ảnh vệ tinh**, nạp ảnh mới hằng ngày, nhưng **tìm kiếm không thường xuyên (infrequently)** — chỉ khi nhà phân tích cần đối chiếu. Yêu cầu **rẻ nhất**, hiệu năng tìm kiếm tốt, **không phải quản lý hạ tầng**.

Giải pháp nào?

- **A.** Lưu vector trong OpenSearch Serverless, dùng vector search.
- **B.** Lưu vector trong DynamoDB, tự viết logic similarity bằng Lambda.
- **C.** Tạo **Amazon S3 vector bucket** với vector index để lưu embedding và tìm kiếm tương đồng.
- **D.** Lưu vector trong RDS for PostgreSQL + pgvector.

<details>
<summary>📖 Xem phân tích & đáp án</summary>

### 🎯 Câu này test concept gì
Chọn kho vector **rẻ nhất** cho khối lượng lớn nhưng **truy vấn thưa** — không phải kho luôn-bật tốn phí duy trì. Thuộc Domain 1 & 4.

### 🔍 Phân tích cốt lõi — từ khóa chết chóc "infrequently"
- Nếu thuê một **database luôn bật** (RDS) hoặc một hệ thống phải **duy trì năng lực tính toán ngầm** (OpenSearch Serverless), bạn trả tiền hàng tháng dù cả tuần không ai tìm kiếm → cực lãng phí.
- Yêu cầu thứ hai: **không quản hạ tầng** — không muốn chọn RAM/CPU, không cài cấu hình máy chủ.

**Thuốc đặc trị — Amazon S3 Vectors:** đây là tính năng **serverless thật sự** của S3. Bạn ném hàng triệu vector vào đó; khi không ai tìm kiếm, chỉ tốn phí lưu trữ S3 **rẻ mạt**. Khi có nhà phân tích bấm "Tìm kiếm", AWS mới tính phí đúng cú click đó (**pay-as-you-go**) → hoàn hảo cho bài toán **tần suất thấp + chi phí rẻ nhất**.

### 🧩 Mổ xẻ từng đáp án
- **C — ĐÚNG.** S3 Vectors là tính năng managed (không quản hạ tầng), mở rộng tới **hàng tỷ vector**. Nhờ bản chất **pay-as-you-go** của S3, nó đặc biệt tiết kiệm cho hệ thống **infrequent searches** — bạn không phải trả tiền "nuôi" máy chủ lúc rảnh.
- **A — SAI (phí duy trì base capacity).** OpenSearch Serverless tìm kiếm vector k-NN cực mạnh, nhưng thiết kế cho hệ thống **tìm kiếm liên tục, tần suất cao**. Dù gọi "serverless", nó vẫn tính **OCU (OpenSearch Compute Units) tối thiểu** để luôn sẵn sàng → ít tìm kiếm thì lãng phí lượng OCU chạy ngầm.
- **B — SAI (sai công cụ).** DynamoDB là Key-Value tuyệt vời nhưng **không có vector search tích hợp**. Cố dùng Lambda quét 80 triệu bản ghi và tự viết toán đo khoảng cách → chậm như rùa, tốn khủng phí tính toán Lambda.
- **D — SAI (phải quản server + phí duy trì).** RDS + pgvector bắt **provision** một DB server (db.m5.large…), trả tiền **từng giờ dù rảnh** + lo bảo trì → vi phạm cả 2 yêu cầu.

### ✅ Kết luận: **C.**

### 💡 Bỏ túi mẹo đi thi
1. **"Infrequently" + "Cost-effective" + "Vector Search"** → tìm ngay giải pháp lưu trữ rẻ có tích hợp vector: **Amazon S3 Vectors**.
2. **Loại bỏ database luôn bật:** mạnh dạn gạch RDS, Aurora, OpenSearch (kể cả Serverless) nếu đề nhấn "thỉnh thoảng mới có người dùng". Các dịch vụ này chỉ rẻ khi hệ thống có **lưu lượng truy cập liên tục / tần suất cao**.

</details>

---

## Câu 17 — Biết luật guardrail nào đã chặn

> Vào vai **"cảnh sát điều tra AI"**: khi hệ thống từ chối một khách hàng, làm sao biết chính xác nó chặn **vì lý do gì** (khách nói bậy, hỏi chủ đề cấm, hay lộ số CMND)?

Một trợ lý AI trên Bedrock có nhiều guardrail (prompt injection, lọc thông tin nhạy cảm, chặn chủ đề cấm). Khi một truy vấn bị chặn, lập trình viên cần **phân tích chi tiết quy tắc guardrail cụ thể nào đã kích hoạt** để tinh chỉnh và phân biệt truy vấn hợp lệ với đe dọa thật.

Cấu hình nào cho phân tích chi tiết nhất?

- **A.** Bật Bedrock model evaluation với job đánh giá tự động + guardrail assessment metrics.
- **B.** Bật model invocation logging, đặt CloudWatch alarm trên `InvocationsIntervened` lọc theo `GuardrailContentSource`.
- **C.** Bật **guardrail tracing** (`{"trace": "enabled"}`), giám sát `InvocationsIntervened` lọc theo **`GuardrailContentSource`**.
- **D.** Bật **guardrail tracing** (`{"trace": "enabled"}`), giám sát `InvocationsIntervened` lọc theo **`GuardrailPolicyType`** (ContentPolicy, TopicPolicy, SensitiveInformationPolicy).

<details>
<summary>📖 Xem phân tích & đáp án</summary>

### 🎯 Câu này test concept gì
Gỡ lỗi/giám sát Guardrails: bật **trace** + lọc đúng **dimension** để biết **luật nào** đã can thiệp. Thuộc Domain 3.

### 🔍 Phân tích cốt lõi — bài toán "tìm kẻ ra tay"
Guardrails của Bedrock như một **trạm kiểm soát nhiều lớp bảo vệ** (nhiều "bảo vệ" khác nhau):
1. Anh bảo vệ soi **"từ ngữ độc hại"** (Content Policy).
2. Anh bảo vệ soi **"chủ đề cấm — vd chính trị"** (Topic Policy).
3. Anh bảo vệ soi **"lộ dữ liệu cá nhân PII"** (Sensitive Information Policy).

Khi một khách bị "đá" văng, bạn cần biết chính xác **anh bảo vệ nào ra tay**. Để làm được: (1) bật **"truy vết" (tracing)** để ghi lại nhật ký kiểm duyệt; (2) lọc nhật ký theo **loại chính sách (Policy Type)** để biết đúng tên lớp bảo vệ đã can thiệp.

### 🧩 Mổ xẻ từng đáp án
- **D — ĐÚNG.** `{"trace": "enabled"}` là lệnh bắt buộc để hệ thống nhả báo cáo chi tiết. Quan trọng nhất, lọc theo **`GuardrailPolicyType`** cho biết chính xác **luật nào bị vi phạm** (ContentPolicy / TopicPolicy / SensitiveInformationPolicy) → lập trình viên mới biết đường tinh chỉnh (fine-tune).
- **C — SAI (tìm sai trọng tâm).** Cũng bật tracing, nhưng lọc theo **`GuardrailContentSource`** — chiều này chỉ cho biết nội dung bị chặn ở **input (câu hỏi)** hay **output (câu trả lời)**, **không** cho biết vi phạm **luật gì**.
- **B — SAI (thiếu tracing).** Model invocation logging ghi input/output nhưng **không cung cấp cấu trúc JSON chi tiết** về quyết định bên trong của Guardrails; lại tiếp tục lọc sai chiều như C.
- **A — SAI (sai công cụ, sai thời điểm).** Model Evaluation dùng ở giai đoạn **testing/pre-production** để chấm điểm độ thông minh của model, **không** phải công cụ theo dõi/gỡ lỗi truy vấn **real-time** trên production.

### ✅ Kết luận: **D.**

### 💡 Bỏ túi mẹo đi thi
1. Muốn biết hệ thống chặn do **vi phạm luật gì** → `GuardrailPolicyType` (Topic / Content / PII).
2. Muốn biết hệ thống chặn ở **câu hỏi hay câu trả lời** → `GuardrailContentSource` (Input / Output).
3. Luôn phải có **`{"trace": "enabled"}`** trong cấu hình API Converse để lấy được phân tích chi tiết.

</details>

---

## Câu 18 — Xác thực an toàn, liên kết IdP, không chứng chỉ dài hạn (Chọn HAI)

> Kiến thức tối quan trọng cấp doanh nghiệp: cho nhân viên/ứng dụng dùng AI an toàn mà **không sợ lộ mật khẩu hay chìa khóa truy cập** — bằng **"tấm thẻ vào cổng dùng một lần"**.

Một công ty cần xác thực an toàn cho ứng dụng bên thứ ba dùng Bedrock. Giải pháp phải: tích hợp **IdP (Identity Provider) hiện có**, giữ **audit log toàn diện**, **loại bỏ chứng chỉ dài hạn (long-lived credentials)**, cấp **quyền truy cập tạm thời** tới Bedrock.

Hai giải pháp nào? (Chọn HAI.)

- **A.** Tích hợp **OIDC với Amazon Cognito**: xác thực qua IdP rồi đổi token lấy chứng chỉ AWS tạm thời để truy cập Bedrock.
- **B.** Lambda authorizer ở API Gateway kiểm tra với LDAP rồi tự phát hành JWT cho truy cập Bedrock.
- **C.** Tạo IAM user cho từng nhân viên, gán quyền qua IAM policy, xoay vòng bằng Secrets Manager.
- **D.** Tạo IAM role + federation qua STS AssumeRole, **lưu chứng chỉ IAM user của ứng dụng trong file cấu hình**.
- **E.** Triển khai **AWS IAM Identity Center với SAML federation** tới IdP, cấu hình permission set cấp quyền Bedrock.

<details>
<summary>📖 Xem phân tích & đáp án</summary>

### 🎯 Câu này test concept gì
**Identity Federation** cấp **chứng chỉ tạm thời** từ IdP, loại bỏ key tĩnh. Thuộc Domain 3.

### 🔍 Phân tích cốt lõi — bài toán "tấm thẻ vào cổng dùng 1 lần"
Trong bảo mật đám mây, kẻ thù nguy hiểm nhất là **long-lived credentials** (Access Key & Secret Key vĩnh viễn). Nếu bạn tạo chìa khóa vĩnh viễn và nhét vào mã nguồn, lỡ hacker trộm được mã nguồn → hệ thống bị xâm nhập **mãi mãi**.

**Thuốc đặc trị — Identity Federation:** cách chuẩn mực là **không tạo tài khoản nội bộ trên AWS**, mà kết nối AWS với hệ thống nhân sự sẵn có (IdP như Okta, Microsoft Entra ID). Khi nhân viên đăng nhập IdP thành công, AWS cấp một **thẻ tạm thời (temporary credentials)** chỉ giá trị vài giờ — hết giờ tự hủy. Không có key tĩnh nào lưu lại, hacker trộm thẻ cũng vô dụng.

### 🧩 Mổ xẻ từng đáp án
- **A — ĐÚNG (cho ứng dụng).** Amazon Cognito quản đăng nhập cho **ứng dụng (web/app)**, hỗ trợ chuẩn **OIDC** liên kết IdP. Điểm ăn tiền: Cognito Identity Pools **đổi token IdP lấy Access Key tạm thời của AWS** để gọi Bedrock; mọi hành động tự được CloudTrail log.
- **E — ĐÚNG (cho nhân sự nội bộ).** **AWS IAM Identity Center** (tên cũ AWS SSO) là công cụ tối thượng để **federation qua SAML 2.0** với hệ thống danh tính công ty: cấp quyền hoàn toàn bằng **chứng chỉ tạm thời** (loại bỏ long-lived credentials), mọi đăng nhập/gọi API đều log trên CloudTrail.
- **B — SAI (chế lại bánh xe, kém an toàn).** Tự viết Lambda nối **LDAP** cũ kỹ rồi tự phát **JWT** là giải pháp custom đầy rủi ro; Bedrock API **không nhận JWT tùy chỉnh trực tiếp** mà vẫn cần đổi sang AWS credentials.
- **C — SAI (vi phạm cốt lõi).** Tạo **IAM Users** cho từng người = tạo **long-lived credentials** (mật khẩu & Access Key vĩnh viễn) — anti-pattern khét tiếng, vi phạm trực tiếp "loại bỏ chứng chỉ dài hạn". Rotation bằng Secrets Manager không đổi bản chất key tĩnh.
- **D — SAI (lỗi bảo mật nghiêm trọng).** Nửa đầu nhắc STS AssumeRole (tạo chứng chỉ tạm thời) là đúng, nhưng vế sau xúi **"lưu chứng chỉ IAM user trong file cấu hình"** — hardcode key tĩnh vào ứng dụng là **lỗi chết người**, vi phạm hoàn toàn yêu cầu.

### ✅ Kết luận: **A + E.**

### 💡 Bỏ túi mẹo đi thi
1. **"Eliminate long-lived credentials"** → gạch ngay mọi đáp án tạo **IAM Users** hoặc lưu **Access Keys**.
2. **2 dịch vụ federation cấp thẻ tạm thời:** **AWS IAM Identity Center (SAML)** cho **nhân sự nội bộ (workforce)** truy cập AWS; **Amazon Cognito (OIDC/SAML)** cho **ứng dụng hướng người dùng (customer-facing)**. Cả hai đều đúng cho dạng bài bảo mật này.

</details>

---

## Câu 19 — Throttling giờ cao điểm, cùng FM, rẻ nhất

> "Căn bệnh" của các hệ thống quá thành công: **tắc đường giờ cao điểm (throttling)**. Làm sao mở rộng giới hạn phục vụ của AI mà **không phải trả một đống tiền** hay tự viết code cân bằng tải?

Một sàn thương mại điện tử dùng Bedrock sinh mô tả sản phẩm, ứng dụng nằm ở **một Region**. Giờ cao điểm bị lỗi "Too many requests, please wait before trying again". Cần **tăng throughput** lúc cao điểm **không thêm overhead vận hành**, **giữ tương thích API Bedrock hiện tại**, **dùng đúng cùng một FM**. Yêu cầu **rẻ nhất**.

Giải pháp nào?

- **A.** Lambda gọi Bedrock với Region gốc mặc định, fallback sang Region phụ khi lỗi.
- **B.** Dùng **Cross-Region Inference** phân phối lưu lượng qua nhiều Region trong một vùng địa lý.
- **C.** Dùng prompt routing phân phối lưu lượng qua nhiều FM cùng họ.
- **D.** Dùng Provisioned Throughput cấp mức throughput cao hơn cho FM.

<details>
<summary>📖 Xem phân tích & đáp án</summary>

### 🎯 Câu này test concept gì
Vượt giới hạn quota lúc cao điểm bằng **Cross-Region Inference**, on-demand, không đổi code. Thuộc Domain 1 & 4.

### 🔍 Phân tích cốt lõi — bài toán "tắc đường giờ cao điểm"
Mỗi Region (vd `us-east-1`) đều có **hạn mức (quota)** số API gọi mỗi phút. Khi có đợt Flash Sale, lượng yêu cầu tăng vọt, Bedrock ở Region đó nghẽn → báo lỗi rate limit / throttling.

**Thuốc đặc trị — Cross-Region Inference:** thay vì bắt lập trình viên tự viết code phức tạp kiểu "nếu Region A lỗi thì gọi Region B", Bedrock có tính năng này: chỉ cần đổi **Model ID** sang **Inference Profile ARN**, AWS tự đứng ra làm cảnh sát giao thông — *"Region A đang tắc, tôi tự đẩy yêu cầu sang Region B (đang rảnh), trả kết quả về, giá vẫn không đổi!"*. Hệ thống mở rộng gấp đôi quota mà **không tốn một dòng code vận hành**.

### 🧩 Mổ xẻ từng đáp án
- **B — ĐÚNG.** Cross-Region Inference thỏa mãn mọi yêu cầu: tránh throttling nhờ gộp quota nhiều Region; vẫn **dùng đúng một FM**; **không** đổi code phức tạp/quản hạ tầng; trả theo **token thực dùng (on-demand)** → **rẻ nhất** cho đợt tăng tải đột biến.
- **D — SAI (giải pháp "đại gia", cực đắt).** Provisioned Throughput nghĩa là **bao trọn gói** một lượng GPU lớn chạy suốt tháng. Nó giải quyết throttling nhưng **chi phí cực cao** (hàng chục ngàn USD/tháng). Với công ty chỉ tắc đường vào **giờ cao điểm**, mua đứt máy chủ 24/7 là lãng phí khủng khiếp, ngược "cost-effective".
- **A — SAI (chế lại bánh xe).** Tự viết Lambda Try/Catch rồi failover sang Region khác chính là cách người ta làm **TRƯỚC KHI** có Cross-Region Inference; phải duy trì code, theo dõi lỗi → phát sinh **overhead vận hành**.
- **C — SAI (vi phạm yêu cầu kỹ thuật).** Intelligent Prompt Routing điều hướng theo **độ khó** câu hỏi (câu dễ → Haiku cho rẻ, câu khó → Sonnet). Nhược điểm: bắt dùng **nhiều FM khác nhau** → vi phạm trắng trợn yêu cầu **"phải dùng CÙNG MỘT FM"**.

### ✅ Kết luận: **B.**

### 💡 Bỏ túi mẹo đi thi
1. **"Too many requests / throttling limits / peak spikes" + "cost-effective"** → giải pháp luôn là **Cross-Region Inference** (mở rộng quota linh hoạt, trả theo dùng).
2. **Provisioned Throughput:** chỉ chọn khi đề nhấn tải **cao, ổn định, liên tục 24/7** (consistent, predictable workload) và sẵn sàng cam kết dài hạn. Có chữ **"spikes"** hay **"peak periods"** thì tuyệt đối **không** chọn Provisioned Throughput.

</details>

---

## Câu 20 — Tẩy PII trước khi đưa vào tìm kiếm

> Bài toán nhạy cảm của ngành tài chính/ngân hàng: **xử lý PII (Personally Identifiable Information — dữ liệu cá nhân)** trước khi đưa vào AI. Làm sao **"tẩy sạch"** số điện thoại, số tài khoản, tên khách trong hàng triệu email cũ trước khi cho AI đọc?

Một ngân hàng muốn xây app di động hỗ trợ truy vấn thông tin tài khoản, dùng kho **email** trao đổi giữa khách và nhân viên (lưu S3) làm nguồn. Dữ liệu chứa **PII không được phép xuất hiện trong kết quả tìm kiếm**.

Giải pháp nào?

- **A.** Dùng Kendra tìm kiếm email trong S3, tích hợp Bedrock FM, dùng **system prompt** để loại PII lúc xử lý truy vấn.
- **B.** Dùng **Amazon Comprehend** phát hiện + **redact PII (bôi đen/ẩn)** từ email trong S3, tích hợp với **Amazon Kendra** để tìm kiếm trên dữ liệu đã xử lý.
- **C.** Dùng Textract trích văn bản, Macie quét PII trong S3, tích hợp Kendra để tìm kiếm.
- **D.** Dùng Comprehend redact PII, tích hợp **DocumentDB** để truy vấn tìm kiếm.

<details>
<summary>📖 Xem phân tích & đáp án</summary>

### 🎯 Câu này test concept gì
Kiến trúc **làm sạch trước, tìm kiếm sau**: redact PII (Comprehend) → index để tìm kiếm (Kendra). Thuộc Domain 3.

### 🔍 Phân tích cốt lõi — bài toán "tẩy rửa dữ liệu"
Hồ sơ email chứa vô số dữ liệu nhạy cảm (tên, số thẻ, địa chỉ). **Nguyên tắc bảo mật số 1** (nhất là ngành tài chính): **không bao giờ đưa raw data chưa làm sạch vào hệ thống tìm kiếm hoặc AI**. Phải có một "máy giặt" để **redact (bôi đen)** toàn bộ thông tin này (ví dụ đổi "Tài khoản 123456" → "Tài khoản [ACCOUNT_NUMBER]").

**Thuốc đặc trị — Comprehend + Kendra:**
1. **Amazon Comprehend (cái máy giặt):** dịch vụ xử lý ngôn ngữ tự nhiên (NLP) có tính năng **PII Redaction** — đọc hàng triệu email, tự tìm số thẻ/tên người và thay bằng `***` hoặc nhãn ẩn danh.
2. **Amazon Kendra (tủ tài liệu thông minh):** sau khi Comprehend "giặt sạch", ném dữ liệu sạch vào Kendra — cỗ máy **enterprise search ngữ nghĩa** cực mạnh. Người dùng tìm thông tin mà tuyệt đối không lo lộ PII của người khác.

### 🧩 Mổ xẻ từng đáp án
- **B — ĐÚNG.** Comprehend (phát hiện + redact PII) + Kendra (lập chỉ mục + enterprise search) là giải pháp hoàn hảo, an toàn và hoàn toàn tự động.
- **A — SAI (sai lầm chết người về bảo mật).** Phó mặc **system prompt** để bảo vệ PII cực rủi ro: LLM không phải bộ lọc xác định tuyệt đối, hoàn toàn có thể bị **jailbreak** hoặc ảo giác và vô tình lộ thông tin. Dữ liệu nhạy cảm phải bị **redact TRƯỚC KHI** AI kịp nhìn thấy.
- **C — SAI (sai toàn bộ công cụ).** **Textract** là OCR đọc chữ từ ảnh/PDF scan — email vốn đã là text thuần, dùng Textract vô nghĩa. **Macie** là công cụ **kiểm toán**: nó chỉ **cảnh báo** "bucket này có PII", **không hề redact** hay biến đổi file cho AI.
- **D — SAI (tìm kiếm sai chỗ).** Nửa đầu dùng Comprehend đúng, nhưng nửa sau dùng **DocumentDB** (DB tương thích MongoDB, lưu JSON) làm công cụ tìm kiếm. DocumentDB **không có khả năng hiểu ngôn ngữ tự nhiên** để người dùng chat tìm kiếm như Kendra → sai kiến trúc.

### ✅ Kết luận: **B.**

### 💡 Bỏ túi mẹo đi thi (cheat sheet công cụ)
- **Amazon Comprehend** — NLP: công cụ **duy nhất** đọc text và **Redact PII**.
- **Amazon Kendra** — enterprise search ngữ nghĩa: tìm tài liệu nội bộ bằng ngôn ngữ tự nhiên.
- **Amazon Macie** — đánh giá/khám phá bảo mật: chỉ **cảnh báo** có PII, **không redact**.
- **Amazon Textract** — OCR cho ảnh/PDF: **không** dùng cho text thô như email.
- **Amazon DocumentDB** — NoSQL JSON: **không** phải search engine ngôn ngữ tự nhiên.

</details>

---

> **Hết 20 câu.** Đây là nội dung gốc do tác giả viết để luyện concept, không phải đề thi thật. Xem [DISCLAIMER](../../DISCLAIMER.md).
