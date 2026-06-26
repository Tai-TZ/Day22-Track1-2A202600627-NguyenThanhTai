# Case 1 - Support Ticket Triage

## Mục tiêu

Case này giúp học viên luyện 4 câu hỏi nên bật ra ngay khi gặp một AI task:

- Cái gì deterministic và nên chấm bằng code?
- Cái gì cần semantic judgment và nên giao cho LLM judge hoặc human?
- Cái gì high-risk nên cần gate chặt hơn?
- Sai ở đâu thì cần escalation sang người thật?

Chỉ cần thiết kế eval ban đầu, không cần code full system.

Case này nối trực tiếp từ track **AI Customer Support Agent** ở Day 18/19, nhưng đổi góc nhìn từ **thiết kế trải nghiệm** sang **thiết kế eval**.

---

## 1. Bối cảnh

Một công ty SaaS B2B dùng AI để đọc ticket support mới và tạo output triage cho hệ thống nội bộ.

Output này không gửi trực tiếp cho khách hàng, nhưng nó được dùng để:

- phân loại ticket,
- đánh dấu mức độ gấp,
- route đến đúng team,
- quyết định có cần người thật nhảy vào hay không.

Nếu AI route sai, ticket có thể bị trễ, bỏ sót escalation, hoặc đẩy sai sang team không xử lý được.

---

## 2. Workflow logic (ASCII)

```text
Khách hàng gửi ticket hỗ trợ
    ↓
AI đọc:
- tiêu đề
- nội dung ticket
- loại khách hàng
    ↓
Hệ thống phải quyết định:
- đây là loại vấn đề gì?
- mức độ khẩn cấp ra sao?
- có cần người thật xử lý ngay không?
- ticket nên vào hàng của team nào?
    ↓
UI inbox nội bộ hiển thị:
- nhãn loại yêu cầu
- mức độ khẩn
- team phụ trách
- cờ "cần xử lý ngay"
- lý do tóm tắt
    ↓
Nếu khách doanh nghiệp + có dấu hiệu chặn công việc
    ↓
Đẩy lên hàng ưu tiên cao / escalation
```

---

## 3. UI hiển thị dự kiến (ASCII)

```text
+----------------------------------------------------------------+
| Hộp thư hỗ trợ nội bộ                                           |
+----------------------------------------------------------------+
| Ticket: T-002                                                   |
| Khách hàng: Công ty ABC (Enterprise)                            |
| Tiêu đề: Thanh toán lỗi, tài khoản bị khóa                      |
|----------------------------------------------------------------|
| AI gợi ý                                                        |
| - Loại yêu cầu: [ ? ]                                           |
| - Mức độ khẩn: [ ? ]                                            |
| - Team phụ trách: [ ? ]                                         |
| - Cần người xử lý ngay: [ ? ]                                   |
| - Lý do tóm tắt: [ .......................................... ] |
|----------------------------------------------------------------|
| Hàng đợi hiện tại: [ Bình thường ] hoặc [ Ưu tiên cao ]         |
+----------------------------------------------------------------+
```

Học viên cần tự đề xuất output contract tối thiểu phía sau để màn hình này hiển thị được.

---

## 4. Input mẫu

```json
{
  "ticket_id": "T-001",
  "subject": "Cannot login after password reset",
  "message": "I reset my password twice but still cannot log in. This is blocking my work.",
  "customer_tier": "enterprise"
}
```

Một input khác:

```json
{
  "ticket_id": "T-002",
  "subject": "URGENT: payment failed and account disabled",
  "message": "Our team is locked out because your billing system failed. Fix this now.",
  "customer_tier": "enterprise"
}
```

---

## 5. Business rules / operational rules

- Output phải đúng schema và đúng allowed enums.
- `confidence` phải nằm trong khoảng `0-1`.
- Nếu `customer_tier = enterprise` và `urgency` là `high` hoặc `critical`, `requires_human` phải bằng `true`.
- Ticket billing không được route sang `product_team`.
- Ticket có dấu hiệu “blocking work”, “locked out”, hoặc “account disabled” không nên bị đánh `low`.
- `reason_codes` phải phản ánh được nội dung ticket, không được bốc thêm sự thật không có trong input.

---

## 6. Ví dụ full luồng để hình dung nhanh

### Tình huống

Khách hàng doanh nghiệp nhắn vào kênh hỗ trợ:

```text
Chị ơi bên em reset mật khẩu 2 lần rồi mà tài khoản admin vẫn không vào được.
Bên em đang bị chặn công việc từ sáng.
```

### Data mẫu

- `customer_tier`: `enterprise`
- `account_name`: `Công ty Minh Phát Logistics`
- `previous_tickets_7d`: `0`
- `channel`: `Zalo OA`

### Workflow ASCII

```text
Khách nhắn vấn đề đăng nhập
    ↓
AI đọc nội dung + loại khách hàng
    ↓
AI phát hiện tín hiệu:
- login issue
- blocked work
- enterprise customer
    ↓
Hệ thống gợi ý:
- category = technical
- urgency = high hoặc critical
- requires_human = true
- route_to = technical_support
    ↓
UI nội bộ đẩy ticket lên hàng ưu tiên
    ↓
Nhân viên hỗ trợ xem lại rồi tiếp nhận
```

### UI trước khi AI xử lý (ASCII)

```text
+----------------------------------------------------------------+
| Hộp thư hỗ trợ nội bộ                                           |
+----------------------------------------------------------------+
| Ticket: T-115                                                   |
| Kênh: Zalo OA                                                   |
| Khách hàng: Minh Phát Logistics                                 |
|----------------------------------------------------------------|
| Nội dung khách nhắn:                                            |
| "Reset mật khẩu 2 lần rồi mà tài khoản admin vẫn không vào..."  |
|----------------------------------------------------------------|
| AI gợi ý: Chưa có                                               |
+----------------------------------------------------------------+
```

### UI sau khi AI xử lý (ASCII)

```text
+----------------------------------------------------------------+
| Hộp thư hỗ trợ nội bộ                                           |
+----------------------------------------------------------------+
| Ticket: T-115                                                   |
| Khách hàng: Minh Phát Logistics (Enterprise)                    |
|----------------------------------------------------------------|
| AI gợi ý                                                        |
| - Loại yêu cầu: Technical                                       |
| - Mức độ khẩn: High                                             |
| - Team phụ trách: Technical Support                             |
| - Cần người xử lý ngay: Có                                      |
| - Lý do tóm tắt: Lỗi đăng nhập đang chặn công việc              |
| - Hàng đợi: Ưu tiên cao                                         |
+----------------------------------------------------------------+
```

Ví dụ này giúp người đọc hình dung ngay:

- AI đang quyết định gì,
- quyết định nào hiển thị ra UI,
- và sai ở đâu thì ảnh hưởng vận hành.

---

## 7. Seed cases

Đây không phải full dataset. Đây chỉ là 3 seed cases để học viên hình dung phạm vi và failure modes.

### Seed A - Happy path

- `subject`: `Cannot login after password reset`
- Kỳ vọng: `category = technical`, `requires_human = true` nếu urgency đủ cao, route về `technical_support`

### Seed B - Ambiguous / low-info

- `subject`: `Help`
- `message`: `Please help asap`
- Kỳ vọng: AI không nên tự tin gán category quá mạnh; cần `unknown` hoặc route theo hướng cần review

### Seed C - High-risk / escalation

- `subject`: `URGENT: payment failed and account disabled`
- Kỳ vọng: `category = billing`, `urgency = critical`, `requires_human = true`, route về `billing_ops` hoặc `human_escalation`

---

## 8. Bạn phải đề xuất thêm 5 Dataset Edge Cases

Sau khi đọc seed cases ở trên, hãy đề xuất thêm 5 case cần đưa vào reference dataset version đầu.

Không cần nộp một bảng coverage riêng. Hãy chọn 5 case đại diện cho các lát cắt khác nhau, ví dụ: match rõ, thiếu tín hiệu, ambiguity, escalation, và regression.

1. Happy path:
   - Input: `subject = "Cannot login after password reset"`, `message = "Reset password twice, admin account still locked"`, `customer_tier = enterprise`
   - Kỳ vọng: `category = technical`, `urgency = high`, `requires_human = true`, `route_to = technical_support`, `queue_priority = high`
   - Bắt failure: AI gán đúng nhóm xử lý cho case rõ ràng nhưng vẫn escalation đúng vì enterprise + blocking work.

2. Ambiguous input:
   - Input: `subject = "Help"`, `message = "Please help asap"`, `customer_tier = standard`
   - Kỳ vọng: `category = unknown` hoặc `clarification_needed`, `confidence <= 0.6`, `requires_human = true`, không tự route cứng sang team chuyên sâu.
   - Bắt failure: AI over-confident — tự gán category/urgency khi thiếu tín hiệu, dẫn tới route sai hàng.

3. Missing information:
   - Input: `subject = "Invoice question"`, `message = ""` (body rỗng), `customer_tier = enterprise`
   - Kỳ vọng: không đoán nội dung billing cụ thể; `reason_codes` không chứa sự thật không có trong input; route về `billing_ops` chỉ khi có tín hiệu rõ hoặc flag cần human clarify.
   - Bắt failure: AI hallucinate chi tiết từ subject mơ hồ và đẩy ticket vào queue sai mà không kích hoạt review.

4. High-risk / escalation:
   - Input: `subject = "URGENT: payment failed and account disabled"`, `message = "Team locked out because billing failed. Fix now."`, `customer_tier = enterprise`
   - Kỳ vọng: `category = billing`, `urgency = critical`, `requires_human = true`, `route_to = billing_ops` (không phải `product_team` hay `support_l1`), `queue_priority = high`
   - Bắt failure: under-triage — giống mock outcome T-002 (billing bị gán Product/Medium/Không escalation) dù ticket có dấu hiệu chặn công việc.

5. Regression case:
   - Input: ticket billing đã từng pass ở v1 prompt: `subject = "Double charged on invoice INV-4421"`, `customer_tier = standard`
   - Kỳ vọng: `category = billing`, `route_to = billing_ops`, không route sang `product_team` (vi phạm business rule cứng).
   - Bắt failure: thay đổi prompt/model làm regress — ticket billing đơn giản bị đẩy sang team không có quyền xử lý thanh toán.

Với mỗi case, thêm 1 dòng ngắn giải thích:

- case này dùng để bắt failure gì?

---

## 9. Mock outcome để soi

Giả sử trên UI nội bộ, hệ thống hiển thị kết quả gợi ý như sau cho `T-002`:

```text
+----------------------------------------------------------------+
| Ticket: T-002                                                   |
| Khách hàng: Enterprise                                          |
|----------------------------------------------------------------|
| AI gợi ý                                                        |
| - Loại yêu cầu: Product question                                |
| - Mức độ khẩn: Medium                                           |
| - Team phụ trách: Support L1                                    |
| - Cần người xử lý ngay: Không                                   |
| - Lý do tóm tắt: Có vấn đề thanh toán                           |
| - Độ tin cậy: 0.91                                              |
+----------------------------------------------------------------+
```

Kết quả này trông có vẻ “ổn” nếu chỉ nhìn bề mặt, nhưng khả năng cao là sai về judgment vận hành.

---

## 10. Nhiệm vụ học viên

Hãy điền workbook bên dưới cho case này.

Không cần:

- viết eval runner,
- viết prompt judge thật,
- làm lại `User Input Grid` đầy đủ như bài test inputs hôm trước,
- tạo full dataset lớn,
- code full system.

Cần làm:

- chọn đúng nguồn chấm cho từng thành phần,
- viết các rule kiểm tra đủ cụ thể để có thể implement sau,
- đặt release gate có ý nghĩa vận hành,
- đề xuất 5 edge cases cần đưa vào reference dataset,
- và lập một pilot plan có thời gian + chi phí sơ bộ.

---

## 11. Bạn nên làm gì ở case 1?

Đây là case scaffold cao, nên cách làm tốt nhất là:

1. Đọc ví dụ full luồng trước để hiểu “một output tốt trông như thế nào”.
2. So mock outcome với ví dụ full luồng để thấy lỗi đang nằm ở đâu.
3. Nhìn từ UI để suy ra các field tối thiểu hệ thống phải có.
4. Điền `Eval Decision Map` trước, rồi mới quay lại viết các kiểm tra tự động và gate.

Case này thường **không bắt buộc phải có domain expert chuyên sâu**. Nếu chọn không cần expert, bạn vẫn phải giải thích vì sao human review vận hành là đủ.

---

## 12. Workbook

Lưu ý chung cho toàn bộ câu trả lời:

- Không chỉ điền đáp án ngắn.
- Với mỗi phần, hãy nêu cả **quyết định** và **lý do**.
- Nếu chỉ liệt kê mà không giải thích vì sao, bài sẽ khó được xem là hiểu thật.

### 1. Unit of Work

- AI đang thực hiện công việc gì?
- Output cuối cùng được dùng bởi ai?
- Nếu sai, hậu quả vận hành là gì?

Gợi ý từ bài hôm trước:

- Đừng chọn “toàn bộ hệ thống hỗ trợ khách hàng”.
- Ở case này, một `Unit of Work` tốt thường là: **một ticket đi vào -> AI gán nhãn, đánh mức ưu tiên, đề xuất route và cờ escalation**.

**Trả lời của bạn:**

Hãy viết 2-4 câu, trong đó có cả:

- bạn chọn lát cắt nào,
- và vì sao đây là đơn vị đủ nhỏ để eval.

> Lát cắt tôi chọn: **một ticket mới đi vào hệ thống → AI đọc subject/message/customer_tier → sinh output triage** (category, urgency, route_to, requires_human, reason tóm tắt, queue priority).
>
> Đây là đơn vị đủ nhỏ vì mỗi ticket là một quyết định độc lập, có input/output rõ ràng, và có thể chấm pass/fail theo từng field. Nó cũng đủ lớn để chạm đúng rủi ro vận hành: route sai hàng, bỏ sót escalation, hoặc under-triage khách enterprise.
>
> Output được dùng bởi nhân viên support trong inbox nội bộ để quyết định ai xử lý và mức ưu tiên. Nếu sai, ticket enterprise bị chậm SLA, billing issue vào team kỹ thuật, hoặc ticket critical nằm ở hàng bình thường — khách mất trust dù AI không trả lời trực tiếp cho họ.

### 2. Quality Question

Viết một câu hỏi chất lượng đủ cụ thể cho lát cắt này.

Gợi ý:

- Đừng hỏi kiểu quá rộng như: “AI có triage tốt không?”
- Nếu AI làm sai ở đây, điều gì sẽ khiến khách hàng mất trust hoặc không hoàn thành mục tiêu?
- Behavior nào là bắt buộc?
- Behavior nào là bị cấm?
- Viết theo dạng: **AI có gắn đúng route và escalation để ticket không bị đi sai hàng xử lý không?**

**Trả lời của bạn:**

Hãy viết 2-4 câu, trong đó có cả:

- câu hỏi chất lượng bạn chọn,
- và vì sao nếu fail ở đây thì ticket sẽ đi sai hoặc gây mất trust.

> **Câu hỏi chất lượng:** Với mỗi ticket đầu vào, AI có gán đúng category và route_to, đặt urgency/requires_human phù hợp với mức độ chặn công việc và tier khách hàng, để ticket không bị under-triage hoặc đẩy sai team không?
>
> **Behavior bắt buộc:** billing → `billing_ops`; enterprise + high/critical → `requires_human = true`; dấu hiệu "locked out"/"blocking work" → urgency không thấp hơn `high`.
>
> **Behavior bị cấm:** billing route sang `product_team`; gán urgency thấp cho ticket enterprise đang bị chặn công việc; reason tóm tắt bốc thêm sự thật không có trong ticket.
>
> Fail ở đây thì ticket critical (như T-002) có thể nằm ở Support L1 với cờ "không cần người" — nhìn UI thì có vẻ ổn (confidence 0.91) nhưng thực tế khách enterprise bị khóa tài khoản chờ quá lâu, mất trust và vi phạm SLA.

### 3. Output Contract tối thiểu

Không cần đoán full JSON hoàn chỉnh. Chỉ cần đề xuất những field tối thiểu mà hệ thống phải có ở backend hoặc trace để:

- render UI ở trên,
- route đúng hàng xử lý,
- trigger escalation nếu cần,
- và chạy eval sau này.

Mẹo lấy từ ví dụ full luồng:

- Hãy nhìn ngược từ UI và mock outcome.
- Field nào không làm thay đổi màn hình, routing hoặc gate thì chưa cần đưa vào.

**Trả lời của bạn:**

Đừng chỉ liệt kê field. Với mỗi field bạn giữ lại, hãy giải thích ngắn vì sao nó cần cho UI, routing, escalation, hoặc eval.

> | Field | Vì sao cần |
> | --- | --- |
> | `ticket_id` | Liên kết output với ticket gốc; bắt buộc cho trace, eval, và hiển thị trên UI. |
> | `category` | Enum (`technical`, `billing`, `feature_request`, `clarification_needed`, `unknown`) — render nhãn "Loại yêu cầu" và là input cho rule route. |
> | `urgency` | Enum (`low`, `medium`, `high`, `critical`) — render "Mức độ khẩn", kích hoạt rule enterprise escalation, và quyết định queue. |
> | `route_to` | Enum (`technical_support`, `billing_ops`, `product_team`, `support_l1`, `human_escalation`) — quyết định team nhận ticket; field quan trọng nhất cho eval routing. |
> | `requires_human` | Boolean — render "Cần người xử lý ngay"; trigger escalation workflow khi `true`. |
> | `reason_summary` | String ngắn — hiển thị trên UI để agent hiểu nhanh; dùng cho LLM judge kiểm tra groundedness. |
> | `reason_codes` | Mảng tag có cấu trúc (vd: `login_issue`, `blocking_work`, `payment_failed`) — dùng cho code rule (vd: có `blocking_work` thì urgency không được `low`) và eval reproducible. |
> | `confidence` | Float 0–1 — hiển thị trên UI; dùng gate: confidence thấp + category không chắc → bắt buộc human review trước khi auto-route. |
> | `queue_priority` | Enum (`normal`, `high`) — render "Hàng đợi"; derive từ urgency + customer_tier + requires_human để sort inbox. |
>
> Không đưa vào contract v0: `sentiment` (không ảnh hưởng routing trong case này), `suggested_reply` (AI không trả lời khách), `internal_notes` dài (không cần cho gate).

### 4. Eval Decision Map

Ở phần này, bạn phải **tự quyết định** đâu là các thành phần thật sự cần chấm.

Đừng chép lại toàn bộ business rules hay toàn bộ UI. Hãy chọn ra những thành phần quan trọng nhất, bám vào:

- `Output Contract` bạn đã đề xuất
- quyết định nào thật sự làm thay đổi route, escalation, hoặc safety
- chỗ nào nếu sai sẽ gây hậu quả vận hành rõ ràng

| Thành phần cần chấm | Code | LLM | Human | Expert | Lý do |
| --- | ---: | ---: | ---: | ---: | --- |
| Schema + enum hợp lệ | ✓ | | | | Deterministic: field bắt buộc, enum cố định, confidence trong [0,1] — validator chạy nhanh trong CI. |
| Rule route billing ≠ product_team | ✓ | | | | Business rule cứng, không cần đọc hiểu ngữ nghĩa — assert `category=billing → route_to≠product_team`. |
| Rule enterprise + high/critical → requires_human | ✓ | | | | Rule vận hành rõ ràng, kết hợp input `customer_tier` và output `urgency` — code check được ngay. |
| Rule blocking signal → urgency không thấp hơn high | ✓ | | ✓ | | Code detect keyword (`locked out`, `blocking work`, `account disabled`); human spot-check vì ngôn ngữ tự nhiên đa dạng (tiếng Việt/Anh). |
| Category đúng với nội dung ticket | | ✓ | ✓ | | Cần hiểu ngữ nghĩa (login vs billing vs feature) — LLM judge scale tốt sau calibration; human label golden set ban đầu. |
| Urgency phù hợp mức độ chặn công việc | | ✓ | ✓ | | Judgment vận hành: "payment failed + locked out" là critical chứ không phải medium — mock T-002 là ví dụ fail ở đây. |
| route_to khớp category + urgency | | ✓ | ✓ | | Cần hiểu mapping nghiệp vụ (billing critical → billing_ops, không phải support_l1) — LLM + human cho case biên. |
| reason_summary grounded (không hallucinate) | | ✓ | ✓ | | Phải đọc hiểu ticket và so với summary — code chỉ check độ dài, không check nội dung đúng/sai. |
| reason_codes phản ánh đúng tín hiệu trong input | | ✓ | | | Semantic: tag `payment_failed` chỉ hợp lệ khi ticket thực sự nhắc payment — LLM judge với rubric cụ thể. |
| Confidence calibration (không over-confident khi ambiguous) | | ✓ | ✓ | | Seed B ("Help asap") cần confidence thấp — judgment về mức chắc chắn; human calibrate ngưỡng ban đầu. |

Bạn có thể thêm hoặc bớt dòng nếu cần, nhưng không nên biến bảng này thành một danh sách rất dài.

Không chấp nhận bảng chỉ tick `Yes/No`. Cột `Lý do` phải nêu được vì sao bạn chọn nguồn chấm đó.

### 5. Kiểm tra tự động bằng code

Liệt kê **đầy đủ** các rule kiểm tra tự động mà bạn cho rằng case này cần có.

Không giới hạn số lượng. Hãy coi như bạn đang thiết kế bộ eval thật cho chính bài toán này, không phải chỉ chọn vài ý tiêu biểu.

Ưu tiên các kiểm tra mà nếu fail thì ticket sẽ đi sai hàng, thiếu escalation, hoặc vỡ schema.

Mỗi ý nên viết theo dạng:

- Kiểm tra: [rule]
  Vì sao nên giao cho code:

- Kiểm tra: Output parse được JSON và có đủ field bắt buộc (`ticket_id`, `category`, `urgency`, `route_to`, `requires_human`, `reason_summary`, `reason_codes`, `confidence`, `queue_priority`).
  Vì sao nên giao cho code: Invariant kỹ thuật — fail thì không render UI và không chạy eval tiếp.

- Kiểm tra: `category` thuộc enum cho phép; `urgency` thuộc enum cho phép; `route_to` thuộc enum cho phép; `queue_priority` ∈ {`normal`, `high`}.
  Vì sao nên giao cho code: Giá trị ngoài enum làm vỡ routing engine — assert tuyệt đối.

- Kiểm tra: `confidence` ∈ [0, 1].
  Vì sao nên giao cho code: Ngưỡng numeric, không cần semantic judgment.

- Kiểm tra: Nếu `category = billing` thì `route_to ≠ product_team`.
  Vì sao nên giao cho code: Business rule cứng từ đề bài — deterministic, không tranh luận.

- Kiểm tra: Nếu input `customer_tier = enterprise` và `urgency ∈ {high, critical}` thì `requires_human = true`.
  Vì sao nên giao cho code: Rule kết hợp input metadata + output field — code đọc cả hai phía.

- Kiểm tra: Nếu `reason_codes` chứa `blocking_work`, `locked_out`, hoặc `account_disabled` thì `urgency ∈ {high, critical}`.
  Vì sao nên giao cho code: Tag có cấu trúc → rule if/else rõ ràng; bắt under-triage kiểu mock T-002.

- Kiểm tra: Nếu `urgency = critical` thì `queue_priority = high`.
  Vì sao nên giao cho code: Derive rule — ticket critical không được nằm hàng bình thường.

- Kiểm tra: Nếu `category = unknown` hoặc `clarification_needed` thì `requires_human = true`.
  Vì sao nên giao cho code: Ticket mơ hồ không được auto-route im lặng — safety invariant.

- Kiểm tra: Nếu `confidence < 0.7` và `category ∈ {unknown, clarification_needed}` thì không auto-route (giữ ở trạng thái chờ review).
  Vì sao nên giao cho code: Ngưỡng numeric + enum — gate trước khi đẩy vào queue.

- Kiểm tra: `reason_summary` không rỗng và độ dài ≥ 10 ký tự.
  Vì sao nên giao cho code: Format check; nội dung đúng/sai giao LLM/human.

- Kiểm tra: `reason_codes` là mảng, mỗi phần tử thuộc taxonomy cho phép (vd: `login_issue`, `payment_failed`, `blocking_work`, …).
  Vì sao nên giao cho code: Ngăn model bịa tag ngoài taxonomy — schema-level check.

- Kiểm tra: `ticket_id` trong output khớp `ticket_id` trong input.
  Vì sao nên giao cho code: Liên kết trace — sai ID là lỗi kỹ thuật nghiêm trọng.

- Kiểm tra (regression): Tập case billing đã pass ở baseline không được đổi `route_to` sang team khác sau prompt change.
  Vì sao nên giao cho code: CI regression suite — so sánh output field với golden snapshot.

### 6. Tiêu chí chấm bằng LLM

Liệt kê **đầy đủ** các tiêu chí semantic mà case này cần có và code không chấm tốt.

Không giới hạn số lượng. Hãy coi như đây là bộ tiêu chí bạn thật sự sẽ dùng để chấm case này.

Chỉ giữ những tiêu chí mà cần đọc hiểu nghĩa của ticket hoặc mức độ hợp lý của lý do tóm tắt.

Mỗi ý nên viết theo dạng:

- Tiêu chí: [criterion]
  Vì sao code không bắt tốt:

- Tiêu chí: **Category accuracy** — `category` có phản ánh đúng loại vấn đề chính trong ticket không (technical vs billing vs feature request)?
  Vì sao code không bắt tốt: Cùng keyword có thể thuộc domain khác nhau; cần đọc hiểu ngữ cảnh cả subject lẫn message.

- Tiêu chí: **Urgency appropriateness** — mức urgency có tương xứng với mức độ chặn công việc và từ ngữ khẩn cấp trong ticket không?
  Vì sao code không bắt tốt: "URGENT" trong subject không luôn là critical, nhưng "locked out + billing failed" có thể là critical dù không có từ URGENT — cần judgment.

- Tiêu chí: **Route correctness** — `route_to` có phù hợp với category và urgency không (vd: billing + critical → billing_ops, không phải support_l1)?
  Vì sao code không bắt tốt: Mapping phụ thuộc kết hợp nhiều field và ngữ nghĩa nghiệp vụ; rule cứng chỉ bắt được vi phạm rõ, không bắt được "gần đúng nhưng sai team".

- Tiêu chí: **Escalation judgment** — `requires_human` có đúng không khi ticket enterprise đang bị chặn công việc hoặc có dấu hiệu account disabled?
  Vì sao code không bắt tốt: Một số case biên (frustrated nhưng không blocking) cần đọc tone và impact thực tế.

- Tiêu chí: **Reason summary groundedness** — `reason_summary` có chỉ nêu sự thật có trong ticket, không thêm chi tiết không được nhắc đến?
  Vì sao code không bắt tốt: Cần so sánh ngữ nghĩa giữa input text và summary — regex không đủ.

- Tiêu chí: **Reason codes precision** — mỗi tag trong `reason_codes` có được support bởi nội dung ticket không?
  Vì sao code không bắt tốt: Tag `payment_failed` có thể match keyword "payment" nhưng ticket thực ra hỏi feature — cần hiểu intent.

- Tiêu chí: **Ambiguity handling** — với input thiếu thông tin (Seed B), AI có tránh gán category quá tự tin và có `confidence` phản ánh uncertainty không?
  Vì sao code không bắt tốt: "Help asap" không có keyword cụ thể — judgment về mức độ chắc chắn là semantic.

- Tiêu chí: **Under-triage detection** — output có bị đánh giá quá thấp so với mức độ nghiêm trọng thực tế không? (đặc biệt case kiểu mock T-002: billing + account disabled nhưng urgency medium, requires_human false)
  Vì sao code không bắt tốt: Từng field riêng lẻ có thể pass rule (category gần đúng, confidence cao) nhưng tổng thể judgment vẫn sai — cần holistic read.

### 7. Human / Expert Review

- Ai cần review?
- Review những case nào?
- Có cần domain expert không? Nếu không, vì sao?

**Trả lời của bạn:**

Đừng chỉ ghi tên team review. Hãy giải thích vì sao đúng nhóm người đó cần xem, và failure nào cần họ xem.

> **Ai review:** Team lead Support / Queue manager (người hiểu routing thực tế và SLA) + 1–2 senior agent từ billing_ops và technical_support.
>
> **Review case nào:**
> - 100% case trong golden set ban đầu (khoảng 80 case) — để tạo label chuẩn cho LLM judge calibration.
> - Mọi case AI đánh `confidence < 0.7` hoặc `category = unknown` — trước khi auto-route.
> - Mọi case enterprise + `requires_human = true` — spot-check 20% trong pilot, 100% trong tuần đầu go-live.
> - Mọi disagreement giữa code rule pass nhưng LLM judge fail (case kiểu mock T-002) — ưu tiên review ngay.
>
> **Có cần domain expert không:** **Không.** Taxonomy category/route và escalation rule đã được định nghĩa rõ bởi team vận hành support SaaS B2B — không có nuance y khoa, pháp lý, hay policy chuyên ngành cần chuyên gia ngoài. Human review từ người vận hành queue hàng ngày đủ để xác nhận judgment nghiệp vụ và calibrate LLM judge.

Nếu chọn **có domain expert**, bạn phải làm thêm 2 phần dưới đây. Nếu **không cần domain expert**, hãy ghi `Không áp dụng` và giải thích 1 câu.

#### 7A. Màn hình cho Domain Expert (ASCII)

Mock một màn hình review cho expert.

Expert cần thấy tối thiểu:

- AI đã route hoặc gắn nhãn gì,
- dấu hiệu hoặc evidence nào khiến case bị đẩy sang expert,
- expert có thể duyệt / sửa / escalation ở đâu.

**Trả lời của bạn:**

```text
Không áp dụng — case ticket triage SaaS B2B không cần domain expert;
human review từ team vận hành support là đủ (xem phần 7).
```

#### 7B. Tiêu chí review của Domain Expert

Liệt kê các tiêu chí domain expert sẽ dùng để duyệt case này.

Không áp dụng.

### 8. Release Gate

Đề xuất release gate phù hợp cho case này. Nêu rõ điều kiện chặn, ngưỡng chất lượng tối thiểu, và trường hợp cần human review.

**Gate 1 — Hard block (CI, chạy trước mọi deploy prompt/model):**

- 100% case pass toàn bộ code rules (schema, enum, business rules cứng).
- 0 regression trên tập billing cases đã pass ở baseline.
- **Chặn deploy nếu fail bất kỳ rule nào** — không có ngoại lệ.

**Gate 2 — Quality threshold (offline eval trên golden set ~80 case):**

| Metric | Ngưỡng tối thiểu | Ghi chú |
| --- | --- | --- |
| Category accuracy | ≥ 90% | So với human label |
| Route correctness | ≥ 88% | Field quan trọng nhất |
| Escalation recall (enterprise + blocking) | ≥ 95% | Ưu tiên không bỏ sót — false negative nguy hiểm hơn false positive |
| Under-triage rate (critical bị gán ≤ medium) | ≤ 2% | Bắt failure kiểu mock T-002 |
| Reason summary groundedness (LLM judge) | ≥ 92% | Không hallucinate |

- **Chặn deploy nếu** escalation recall < 95% hoặc under-triage rate > 2% — dù các metric khác đạt.

**Gate 3 — Runtime safety (production):**

- Ticket `confidence < 0.7` hoặc `category ∈ {unknown, clarification_needed}` → **không auto-route**, giữ ở hàng "Chờ review" cho đến khi human duyệt.
- Ticket enterprise + `urgency ∈ {high, critical}` → luôn hiển thị banner "Ưu tiên — cần xác nhận" trên UI dù AI đã gợi ý.
- Mọi thay đổi prompt/model → chạy lại full offline eval trước khi bật cho > 10% traffic.

**Gate 4 — Human review bắt buộc trước scale:**

- Tuần 1 pilot (10% ticket mới): 100% output AI được spot-check bởi queue manager.
- Chỉ scale lên 50%/100% khi spot-check cho thấy agreement ≥ 90% với human judgment trên 200 ticket thực tế.

**Trường hợp cần human review ngay (không chờ gate):**

- Disagreement giữa code pass và LLM judge fail trên cùng ticket.
- Khách enterprise phản hồi SLA breach trong 24h sau triage AI.
- Bất kỳ ticket có `account disabled` + `requires_human = false` (vi phạm rule hoặc miss detection).

### 9. Kế hoạch chạy thử và dự toán chi phí

Làm phần này với giả định team của bạn vừa nhận đề bài triage này từ công ty.

Bạn là PM phụ trách đề xuất cách xây bộ eval, cách chạy thử, và chi phí cần xin để trả lời câu hỏi:

- hướng làm này hiện chính xác tới đâu
- cần thêm những checkpoint nào trước khi đề xuất triển khai tiếp
- và với một khoản budget thử nghiệm nhỏ, team có thể chứng minh được gì

README của folder này chỉ cho khung tính. Hãy giữ cách tính đơn giản: phần người tính bằng `giờ công`, phần máy tính bằng `chi phí API key` tính từ **giá thật** của model / dịch vụ bạn chọn.

Để làm phần này, bạn cần tự tính và nêu rõ:

- giá API thật bạn dùng để tính
- tổng số cases pilot dự kiến
- tổng số lần chạy / lặp lại dự kiến
- tổng giờ PM / thiết kế eval
- tổng giờ vận hành / kỹ thuật
- tổng giờ human review
- nếu có `domain expert`, tổng số giờ expert
- tổng chi phí API key
- tổng chi phí pilot
- tổng thời gian dự kiến

Có thể lấy mốc tham khảo để nhẩm nhanh:

- khoảng `50-100 cases`
- khoảng `30-50 lần chạy / lặp lại`

Không cần trình bày thành bảng. Hãy tự chọn cách trình bày miễn là người đọc nhìn vào hiểu được bạn đã tính gì và chi phí tổng rơi vào đâu.

Sau phần này, viết thêm 2-4 câu ngắn:

- bạn dùng giá API thật từ đâu để tính,
- với quy mô này chi phí tổng rơi vào khoảng nào,
- và vì sao plan này đủ để chứng minh case có thể pilot được.

**Mục tiêu pilot:** Trả lời 3 câu hỏi — (1) accuracy hiện tại trên 80 case đại diện, (2) checkpoint nào còn thiếu trước khi scale, (3) với ~$1,200 budget team chứng minh được gì.

**Giả định quy mô:**

- 80 pilot cases (gồm 3 seed + 5 edge case tự đề xuất + ~72 case từ ticket thật đã anonymize)
- 40 lần chạy/lặp (4 vòng prompt iteration × 10 lần chạy full suite mỗi vòng)
- Tổng API calls triage: 80 × 40 = **3,200 lần**
- LLM judge calibration: 80 cases × 8 tiêu chí × 2 vòng = **1,280 lần** (chỉ trên golden set, không mỗi iteration)

**Giá API (tham chiếu OpenAI pricing, tháng 6/2025):**

- Triage model: **GPT-4o-mini** — $0.15/1M input tokens, $0.60/1M output tokens ([openai.com/api/pricing](https://openai.com/api/pricing))
- LLM judge: **GPT-4o** — $2.50/1M input, $10.00/1M output (cần judge chính xác hơn cho calibration)
- Ước tính mỗi lần triage: ~700 input tokens + 280 output tokens
- Ước tính mỗi lần judge: ~1,400 input tokens + 120 output tokens

**Chi phí API:**

- Triage: 3,200 × (700×0.15/1M + 280×0.60/1M) = 3,200 × $0.000273 ≈ **$0.87**
- Judge: 1,280 × (1,400×2.50/1M + 120×10/1M) = 1,280 × $0.0047 ≈ **$6.02**
- Buffer 20% cho retry và dev test ≈ **$1.50**
- **Tổng API ≈ $8–10**

**Giờ công người:**

| Vai trò | Giờ | Việc làm |
| --- | --- | --- |
| PM / thiết kế eval | 18h | Định nghĩa rubric, golden set, release gate, báo cáo pilot |
| Engineer | 12h | Setup eval harness nhẹ, code rules, tích hợp API, dashboard kết quả |
| Support lead + senior agents | 10h | Label 80 case, review disagreement, spot-check tuần pilot |
| **Tổng giờ người** | **40h** | |

- Giả định chi phí nội bộ: **$30/giờ** (blended rate team nhỏ tại VN)
- **Chi phí người: 40h × $30 = $1,200**

**Tổng chi phí pilot: ~$1,210** (trong đó API chỉ ~1%, phần lớn là giờ label/review)

**Timeline: 3 tuần**

- Tuần 1: Thu thập 80 case, human label golden set, implement code rules
- Tuần 2: 2 vòng prompt iteration + LLM judge calibration
- Tuần 3: 2 vòng iteration tiếp, chốt release gate, báo cáo kết quả + đề xuất scale 10%

**Pilot chứng minh được gì:**

- Con số accuracy thực trên category, route, escalation recall — không chỉ "trông ổn" như mock T-002
- Bộ code rules + LLM judge đã calibrate có thể chạy trong CI trước mỗi deploy
- Under-triage rate có đo được và có gate chặn (mục tiêu ≤ 2%)
- Chi phí API negligible ở quy mô này — rủi ro chính là judgment sai, không phải chi phí model

**Tóm tắt:**

Giá API lấy từ trang pricing chính thức của OpenAI (GPT-4o-mini cho triage, GPT-4o cho judge). Với 80 case và 40 lần chạy, tổng chi phí pilot khoảng **$1,200**, trong đó API chỉ ~$10 và phần lớn là 40 giờ công label/review. Plan này đủ nhỏ để xin budget nhanh, nhưng đủ cover failure mode nguy hiểm nhất (under-triage enterprise, route billing sai team) để quyết định có pilot 10% traffic hay không.

---
