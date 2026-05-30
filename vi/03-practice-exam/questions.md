# Practice Questions (Original)

[← Về Practice Exam](./README.md)

> Câu hỏi gốc do tác giả tự viết. Không phải đề thi thật. Xem [DISCLAIMER](../../DISCLAIMER.md).

---

## Câu 1 — (mẫu định dạng)

> 👇 Đây là **câu mẫu** để anh duyệt định dạng. Câu thật sẽ được viết từ concept anh cung cấp.

Một công ty bảo hiểm vận hành chatbot hỗ trợ khách hàng dựng trên Amazon Bedrock. Chatbot xử lý hồ sơ bồi thường và thường nhận các đoạn hội thoại có chứa thông tin cá nhân nhạy cảm (số hợp đồng, số điện thoại, địa chỉ). Đội pháp chế yêu cầu: hệ thống **không được để lộ** thông tin cá nhân (PII) trong câu trả lời gửi về phía khách hàng, và phải **chặn các chủ đề ngoài phạm vi** (ví dụ tư vấn pháp lý). Công ty muốn giải pháp với **ít công sức phát triển (custom development) nhất** và quản lý tập trung được.

Giải pháp nào đáp ứng yêu cầu với **ít công sức phát triển nhất**?

- **A.** Tự viết regex và bộ lọc từ khóa trong code ứng dụng để phát hiện và che PII trước khi trả về.
- **B.** Cấu hình **Amazon Bedrock Guardrails** với bộ lọc PII và denied topics, áp dụng cho cả input và output.
- **C.** Dùng Amazon Comprehend để phát hiện PII rồi tự xây service trung gian xóa PII, kết hợp prompt nhắc model tránh chủ đề cấm.
- **D.** Tinh chỉnh (fine-tune) một mô hình riêng để model học cách không xuất PII và từ chối chủ đề ngoài phạm vi.
- **E.** Đặt CloudWatch Logs filter để cảnh báo khi phát hiện PII trong log đầu ra.

<details>
<summary>📖 Xem phân tích & đáp án</summary>

### 🎯 Câu này test concept gì
Sử dụng **managed guardrails** (lan can an toàn được quản lý) để vừa lọc PII vừa chặn chủ đề — thay vì tự xây — nhằm tối thiểu công sức phát triển. Đây là concept thuộc **Domain 3 (AI Safety, Security & Governance)**.

### 🔍 Phân tích yêu cầu
Ba từ khóa quyết định:
1. **"ít công sức phát triển nhất"** → ưu tiên dịch vụ managed, loại bỏ phương án tự code/tự huấn luyện.
2. **"chặn PII ở output" + "chặn chủ đề ngoài phạm vi"** → cần một cơ chế làm được **cả hai** trong một chỗ.
3. **"quản lý tập trung"** → cấu hình tách khỏi code, tái dùng cho nhiều ứng dụng.

### Giải thích từng đáp án
- **A — Sai.** Regex/từ khóa tự viết là công sức phát triển lớn, dễ sót, khó bảo trì; không "ít công sức nhất".
- **B — Đúng.** Amazon Bedrock Guardrails là tính năng managed: bật **PII filter** và **denied topics**, áp cho input lẫn output, cấu hình tập trung, gần như không cần code. Khớp cả ba từ khóa.
- **C — Sai.** Comprehend phát hiện PII tốt nhưng vẫn phải **tự xây service trung gian** + dựa vào prompt để chặn chủ đề (không đáng tin bằng guardrail) → nhiều công sức hơn B.
- **D — Sai.** Fine-tune tốn dữ liệu, chi phí và thời gian lớn; không đảm bảo chặn PII tuyệt đối → ngược với "ít công sức nhất".
- **E — Sai.** CloudWatch chỉ **cảnh báo sau khi đã lộ**, không **ngăn** PII xuất ra cho khách hàng → không đạt yêu cầu.

### ✅ Kết luận
**Đáp án đúng: B.**

### 💡 Bài học
- Khi đề nhấn mạnh "**LEAST development effort / operational overhead**", hãy nghiêng về **managed service** (ở đây là Bedrock Guardrails) thay vì tự build.
- Phân biệt **ngăn chặn (prevent)** vs **phát hiện/cảnh báo (detect/alert)**: yêu cầu "không được để lộ" đòi cơ chế **chặn**, nên loại các phương án chỉ giám sát/log.
- Mô tả ngắn gọn trong 1 câu có thể kèm nhiều yêu cầu (PII + denied topics); ưu tiên giải pháp xử lý **trọn gói** trong một dịch vụ.

</details>

---

<!--
KHUÔN CHO CÂU MỚI (copy phần dưới):

## Câu N — «chủ đề»

«Scenario...»

- **A.** ...
- **B.** ...
- **C.** ...
- **D.** ...
- **E.** ...

<details>
<summary>📖 Xem phân tích & đáp án</summary>

### 🎯 Câu này test concept gì
### 🔍 Phân tích yêu cầu
### Giải thích từng đáp án
### ✅ Kết luận
### 💡 Bài học

</details>
-->
