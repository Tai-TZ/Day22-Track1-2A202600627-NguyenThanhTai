# Case 2 - Sales Chat Copilot: Tóm tắt hội thoại và tra cứu theo tín hiệu khách gửi

## Mục tiêu

Case này đại diện cho một kiểu AI rất hay gặp ở thị trường Việt Nam:

- đội sales hoặc chăm sóc khách hàng đang nhắn tin với khách qua Zalo, Facebook, live chat hoặc CRM inbox,
- AI không thay người chốt đơn hoàn toàn,
- nhưng AI có thể đọc hội thoại, tóm tắt ngữ cảnh, phát hiện tín hiệu như số điện thoại / email / mã đơn / mã khách,
- rồi tra cứu hệ thống nội bộ để đưa thông tin cần thiết cho nhân viên xử lý nhanh hơn.

Điểm khó của case này là:

- có nhiều logic lookup tự động,
- có nguy cơ match sai người hoặc sai đơn,
- có dữ liệu nhạy cảm,
- và rất dễ “trông thông minh” dù thực tế đang suy luận sai.

Chỉ cần thiết kế eval ban đầu, không cần code full system.

---

## 1. Bối cảnh

Một doanh nghiệp bán hàng online tại Việt Nam có đội sales / chăm sóc khách hàng xử lý tin nhắn từ nhiều kênh:

- Zalo OA,
- Facebook Messenger,
- web chat,
- CRM inbox nội bộ.

Khi khách nhắn tin, nhân viên thường phải làm nhiều việc thủ công:

- đọc lại lịch sử hội thoại,
- hiểu khách đang hỏi về vấn đề gì,
- tự tìm số điện thoại / email / mã đơn trong đoạn chat,
- tra CRM hoặc hệ thống đơn hàng,
- rồi mới biết khách này là ai, đang ở trạng thái nào, đơn hàng nào liên quan và nên trả lời tiếp ra sao.

Nhóm muốn thêm một **Sales Chat Copilot** để:

- tóm tắt cuộc hội thoại hiện tại,
- phát hiện các mã hoặc thông tin nhận diện khách,
- tra cứu nhanh hồ sơ khách / đơn hàng / ticket cũ,
- gợi ý thông tin quan trọng cho nhân viên,
- và có thể gợi ý câu trả lời nháp.

AI **không được tự gửi tin nhắn**, **không được tự chốt đơn**, và **không được tự sửa dữ liệu khách**.

---

## 2. Workflow logic tham khảo (ASCII)

```text
Khách nhắn tin đến từ Zalo / Facebook / web chat
    ↓
AI đọc:
- tin nhắn mới nhất
- lịch sử hội thoại gần đây
- metadata kênh nhắn tin
    ↓
AI phát hiện tín hiệu:
- số điện thoại
- email
- mã đơn
- mã khách
- tên sản phẩm / nhu cầu mua
    ↓
Nếu có tín hiệu đủ mạnh
    ↓
Tra CRM / OMS / ticket history
    ↓
Hiển thị cho nhân viên:
- tóm tắt hội thoại
- khách đang hỏi gì
- hồ sơ / đơn hàng liên quan
- cảnh báo nếu có điểm mâu thuẫn
- gợi ý bước tiếp theo
    ↓
Nhân viên xem lại và quyết định:
- trả lời thủ công
- dùng nháp AI rồi sửa
- hỏi thêm khách
- chuyển sale khác / chuyển CSKH / escalate
```

---

## 3. Khung UI gợi ý (ASCII)

```text
+------------------------------------------------------------------------------------------------+
| Hộp chat khách hàng                                                                            |
+------------------------------------------------------------------------------------------------+
| Kênh: Zalo OA                               Khách: Chưa xác định chắc chắn                      |
|-----------------------------------------------------------------------------------------------|
| Khách: Chị kiểm tra giúp em đơn này với, em gửi số 0909123456                                  |
| Khách: Mã đơn hình như là DH-48291                                                             |
| NV sale: ...                                                                                   |
|-----------------------------------------------------------------------------------------------|
| Copilot                                                                                       |
| - Tóm tắt hội thoại: [ ................................................................. ]    |
| - Tín hiệu phát hiện: [ số điện thoại ] [ mã đơn ]                                            |
| - Khách / đơn liên quan: [ ? ]                                                                 |
| - Cảnh báo: [ Có / Không ]                                                                     |
| - Gợi ý bước tiếp theo: [ ............................................................... ]    |
|-----------------------------------------------------------------------------------------------|
| [Xem hồ sơ] [Xem đơn hàng] [Dùng nháp AI] [Hỏi thêm khách] [Chuyển người xử lý]              |
+------------------------------------------------------------------------------------------------+
```

Học viên có thể dùng khung này hoặc chỉnh lại nếu thấy logic sản phẩm cần thay đổi. Điểm quan trọng là phải tự đề xuất output contract tối thiểu phía sau để màn hình này hiển thị được.

---

## 4. Tình huống mẫu

### Tình huống A - Khách gửi số điện thoại

```text
Khách: Chị check giúp em đơn cũ với, số em là 0909123456.
```

### Tình huống B - Khách gửi email

```text
Khách: Bên mình kiểm tra lại giúp mình email linh.nguyen@abc.vn nhé, hôm trước có tư vấn mà chưa thấy báo giá.
```

### Tình huống C - Khách gửi mã đơn

```text
Khách: Em hỏi đơn DH-48291 đang tới đâu rồi ạ?
```

### Tình huống D - Khách hỏi mơ hồ

```text
Khách: Chị ơi bên em xử lý giúp case này với, gấp lắm.
```

---

## 5. Business rules / operational rules

- AI có thể **đề xuất tra cứu** hoặc **tự động tra cứu nội bộ** nếu tín hiệu nhận diện đủ rõ, nhưng không được tự gửi phản hồi cho khách.
- Nếu có nhiều bản ghi cùng khớp với một số điện thoại / email / mã, AI không được tự chốt một bản ghi duy nhất.
- Nếu không tìm thấy hồ sơ phù hợp, AI phải nói rõ là “chưa tìm thấy”, không được bịa ra khách hoặc đơn.
- Nếu khách gửi dữ liệu nhạy cảm, hệ thống chỉ hiển thị phần cần thiết cho nhân viên, không bung toàn bộ dữ liệu không liên quan.
- Nếu kết quả CRM và kết quả đơn hàng mâu thuẫn, AI phải cảnh báo mức độ không chắc chắn.
- AI có thể gợi ý nháp trả lời, nhưng bản nháp phải được nhân viên xem lại trước khi gửi.
- AI không được tự tạo đơn mới chỉ vì phát hiện nhu cầu mua, trừ khi luồng đó được người dùng nội bộ xác nhận rõ.

---

## 6. Ví dụ full luồng để hình dung nhanh

### Tình huống

Khách nhắn vào Zalo OA:

```text
Chị kiểm tra giúp em đơn DH-48291 với ạ.
Số em là 0909123456.
Hôm trước chị tư vấn cho em máy lọc nước rồi.
```

### Data mẫu

**Dữ liệu từ CRM**

- Tên khách: `Nguyễn Minh Linh`
- Số điện thoại: `0909123456`
- Kênh lead: `Zalo OA`
- Sales owner: `Trâm`
- Trạng thái lead: `Đã mua lần 1`

**Dữ liệu từ OMS**

- Mã đơn: `DH-48291`
- Sản phẩm: `Máy lọc nước RO Mini`
- Trạng thái: `Đang giao`
- Dự kiến giao: `Hôm nay`

**Lịch sử hội thoại gần đây**

- Khách từng hỏi báo giá
- Sales đã tư vấn 2 model
- Khách đã chốt 1 model hôm trước

### Workflow ASCII

```text
Khách gửi mã đơn + số điện thoại
    ↓
AI phát hiện 2 tín hiệu tra cứu:
- mã đơn
- số điện thoại
    ↓
AI tra CRM + OMS
    ↓
Hệ thống nối thông tin:
- đây là khách cũ
- đơn DH-48291 đang giao
- sales owner hiện tại là Trâm
    ↓
Copilot hiển thị:
- tóm tắt hội thoại
- hồ sơ khách
- trạng thái đơn
- gợi ý bước tiếp theo
    ↓
Nhân viên sales đọc lại
    ↓
Nhân viên chọn:
- dùng nháp AI rồi sửa
- xem đơn chi tiết
- hỏi thêm khách
```

### UI trước khi AI xử lý (ASCII)

```text
+------------------------------------------------------------------------------------------------+
| Hộp chat khách hàng                                                                            |
+------------------------------------------------------------------------------------------------+
| Kênh: Zalo OA                                                                                  |
|-----------------------------------------------------------------------------------------------|
| Khách: Chị kiểm tra giúp em đơn DH-48291 với ạ.                                               |
| Khách: Số em là 0909123456.                                                                    |
|-----------------------------------------------------------------------------------------------|
| Copilot: Chưa phân tích                                                                        |
+------------------------------------------------------------------------------------------------+
```

### UI sau khi AI xử lý (ASCII)

```text
+------------------------------------------------------------------------------------------------+
| Hộp chat khách hàng                                                                            |
+------------------------------------------------------------------------------------------------+
| Kênh: Zalo OA                               Sales owner hiện tại: Trâm                         |
|-----------------------------------------------------------------------------------------------|
| Khách: Chị kiểm tra giúp em đơn DH-48291 với ạ.                                               |
| Khách: Số em là 0909123456.                                                                    |
|-----------------------------------------------------------------------------------------------|
| Copilot                                                                                       |
| - Tóm tắt hội thoại: Khách cũ đang hỏi về tình trạng đơn vừa mua.                             |
| - Tín hiệu phát hiện: [0909123456] [DH-48291]                                                  |
| - Hồ sơ khách: Nguyễn Minh Linh - Đã mua lần 1                                                 |
| - Đơn hàng: Máy lọc nước RO Mini - Trạng thái: Đang giao                                       |
| - Gợi ý: Báo khách đơn đang giao hôm nay và xác nhận khung giờ nhận hàng.                     |
| - Nháp trả lời: "Dạ em thấy đơn DH-48291 đang được giao hôm nay..."                            |
|-----------------------------------------------------------------------------------------------|
| [Xem CRM] [Xem đơn] [Dùng nháp AI] [Hỏi thêm khách]                                           |
+------------------------------------------------------------------------------------------------+
```

Ví dụ này giúp người đọc hình dung ngay:

- AI đang tự động bắt tín hiệu gì,
- lookup nào đang diễn ra ở hậu trường,
- thông tin nào được kéo ra cho sales,
- và ranh giới giữa “gợi ý” với “tự hành động”.

---

## 7. Seed cases

Đây không phải full dataset. Đây chỉ là các seed cases để học viên hình dung phạm vi và failure modes.

### Seed A - Match rõ, một bản ghi duy nhất

- Khách gửi đúng số điện thoại.
- CRM trả về đúng một hồ sơ.
- Có một đơn hàng gần nhất đang giao.
- Kỳ vọng: AI tóm tắt đúng và hiển thị đúng đơn liên quan.

### Seed B - Một tín hiệu khớp nhiều bản ghi

- Cùng một số điện thoại gắn với hai hồ sơ do nhập trùng.
- Kỳ vọng: AI phải cảnh báo mơ hồ và yêu cầu nhân viên chọn đúng hồ sơ.

### Seed C - Khách gửi mã đơn hợp lệ nhưng không phải khách hiện tại

- Mã đơn tồn tại trong hệ thống nhưng thuộc người khác.
- Kỳ vọng: AI không được suy ra đây chắc chắn là đơn của người đang chat.

### Seed D - Khách hỏi mơ hồ, chưa có tín hiệu tra cứu

- Kỳ vọng: AI không nên bịa hồ sơ; nên gợi ý hỏi thêm số điện thoại / email / mã đơn.

### Seed E - Dữ liệu hệ thống mâu thuẫn

- CRM nói khách đang là lead mới.
- OMS lại có lịch sử đơn cũ.
- Kỳ vọng: AI phải nêu rõ điểm mâu thuẫn thay vì tóm tắt như thể mọi thứ đã chắc chắn.

---

## 8. Mock outcome để soi

Giả sử trên UI nội bộ, Copilot hiển thị như sau sau khi khách gửi:

```text
Khách: Chị kiểm tra giúp em đơn này với, em gửi số 0909123456. Mã đơn DH-48291.
```

```text
+------------------------------------------------------------------------------------------------+
| Copilot                                                                                        |
+------------------------------------------------------------------------------------------------+
| Tóm tắt hội thoại: Khách hỏi về tình trạng đơn hàng cũ.                                        |
| Tín hiệu phát hiện: 0909123456, DH-48291                                                       |
| Khách liên quan: Nguyễn Minh Linh                                                              |
| Đơn liên quan: DH-48291 - Đã giao thành công                                                   |
| Cảnh báo: Không                                                                                |
| Gợi ý cho sales: Báo khách đơn đã giao và mời mua thêm sản phẩm mới.                           |
+------------------------------------------------------------------------------------------------+
```

Kết quả này có thể trông “mượt”, nhưng vẫn có khả năng rất nguy hiểm nếu:

- số điện thoại khớp nhiều người,
- mã đơn thuộc khách khác,
- trạng thái “đã giao” là dữ liệu cũ,
- hoặc AI đang đẩy sales upsell sai thời điểm.

---

## 9. Bộ test gợi ý v0

Phần này chỉ để gợi ý cách nghĩ coverage từ bài hôm trước. Không phải deliverable bắt buộc.

Bạn có thể dùng 8 tình huống dưới đây để nghĩ coverage ban đầu:

| ID | Tình huống | Điều cần bắt |
| --- | --- | --- |
| SC-01 | Khách gửi số điện thoại đúng format, match 1 hồ sơ | lookup đúng |
| SC-02 | Khách gửi email viết hoa/thường lẫn lộn | normalization |
| SC-03 | Khách gửi mã đơn sai 1 ký tự | không tự match bừa |
| SC-04 | Cùng số điện thoại khớp 2 hồ sơ | ambiguity handling |
| SC-05 | Khách chỉ nói “chị kiểm tra giúp em case này” | ask for missing info |
| SC-06 | CRM và OMS mâu thuẫn | uncertainty + warning |
| SC-07 | AI gợi ý nháp trả lời nhưng nhầm intent bán hàng thành hậu mãi | summary / intent error |
| SC-08 | Khách gửi tiếng Việt không dấu + mã đơn | robustness với ngôn ngữ thực tế |

---

## 10. Bạn phải đề xuất thêm 5 Dataset Edge Cases

Sau khi đọc bộ test gợi ý v0 ở trên, hãy đề xuất thêm 5 case cần đưa vào reference dataset version đầu.

Không cần nộp một bảng coverage riêng. Hãy chọn 5 case đại diện cho các lát cắt khác nhau, ví dụ: match rõ, thiếu tín hiệu, ambiguity, dữ liệu mâu thuẫn, và action safety.

1. Happy path:
   - Input: Khách gửi SĐT `0909123456` + mã đơn `DH-48291` · CRM 1 hồ sơ · OMS đơn đang giao, khớp cùng khách
   - Kỳ vọng: `lookup_status = matched`, hiển thị đúng khách + đơn, `ambiguity_warning = false`, `suggested_next_step` phù hợp trạng thái "đang giao"
   - Bắt failure: baseline regression — lookup và summary đúng cho case rõ ràng nhất.

2. Ambiguous lookup:
   - Input: SĐT `0909123456` khớp 2 hồ sơ CRM (nhập trùng)
   - Kỳ vọng: `lookup_status = ambiguous`, `matched_customer_id = null`, `ambiguity_warning = true`, gợi ý nhân viên chọn hồ sơ — không auto-chốt một người
   - Bắt failure: AI tự pick một bản ghi — match nhầm khách, trả lời sai đơn.

3. Missing information:
   - Input: "Chị ơi bên em xử lý giúp case này với, gấp lắm" — không có SĐT/email/mã đơn
   - Kỳ vọng: `lookup_status = insufficient_signal`, không bịa hồ sơ, `suggested_next_step` = hỏi thêm SĐT/mã đơn, `confidence` thấp
   - Bắt failure: Over-confident lookup — Copilot hiển thị khách/đơn giả khi chưa có tín hiệu.

4. Conflicting systems:
   - Input: SĐT khớp CRM (lead mới) nhưng mã đơn `DH-48291` thuộc khách khác · hoặc CRM "lead mới" vs OMS có đơn cũ
   - Kỳ vọng: `conflict_warning = true`, không gộp thành narrative chắc chắn, không gợi ý upsell khi trạng thái đơn chưa xác minh
   - Bắt failure: Mock outcome kiểu "đã giao + mời mua thêm" khi dữ liệu mâu thuẫn hoặc đơn thuộc người khác.

5. Regression case:
   - Input: Mã đơn hợp lệ `DH-48291` nhưng thuộc người khác, khách chat chỉ gửi SĐT của mình (không khớp owner đơn)
   - Kỳ vọng: không gán `matched_order_id` chắc chắn cho khách đang chat; cảnh báo mâu thuẫn SĐT vs owner đơn
   - Bắt failure: Seed C — AI suy luận "chắc chắn là đơn của người đang nhắn" dù ownership không khớp.

Với mỗi case, thêm 1 dòng ngắn giải thích:

- case này dùng để bắt failure gì?

---

## 11. Nhiệm vụ học viên

Hãy điền workbook bên dưới cho case này.

Không cần:

- viết connector CRM thật,
- viết regex hoặc parser thật,
- làm lại `User Input Grid` hay `Scenario Dataset v0/v1`,
- code full workflow,
- dựng UI đẹp.

Cần làm:

- xác định unit of AI work đủ nhỏ,
- viết quality question cho lát cắt này,
- đề xuất output contract tối thiểu,
- quyết định phần nào chấm bằng code / LLM / human,
- đặt release gate cho hành vi tra cứu và hiển thị gợi ý,
- đề xuất edge cases cho dataset,
- và lập pilot plan có thời gian + chi phí sơ bộ.

---

## 12. Bạn nên làm gì ở case 2?

Đây là case scaffold trung bình, nên bạn không cần giữ nguyên workflow và UI gợi ý.

Nên làm theo thứ tự:

1. Dùng workflow tham khảo để xác định các bước chính của hệ thống.
2. Quyết định chỗ nào AI được tự lookup, chỗ nào phải hỏi thêm.
3. Xem lại khung UI và chỉnh nếu bạn thấy thiếu checkpoint quan trọng.
4. Chỉ sau đó mới chốt output contract và release gate.

Case này thường thiên về **ops / sales / CRM** hơn là domain chuyên môn sâu. Nếu chọn **không cần domain expert**, bạn vẫn phải giải thích rõ vì sao chỉ cần human review từ team vận hành là đủ.

---

## 13. Workbook

Lưu ý chung cho toàn bộ câu trả lời:

- Không chỉ điền đáp án ngắn.
- Với mỗi phần, hãy nêu cả **quyết định** và **lý do**.
- Nếu chỉ liệt kê mà không giải thích vì sao, bài sẽ khó được xem là hiểu thật.

### 1. Unit of Work

- AI đang thực hiện công việc gì?
- Output cuối cùng được dùng bởi ai?
- Nếu sai, hậu quả vận hành là gì?

Gợi ý từ bài hôm trước:

- Đừng chọn “toàn bộ Copilot cho sales”.
- Ở case này, một `Unit of Work` tốt thường là: **một tín hiệu khách gửi vào -> AI phát hiện tín hiệu -> tra cứu -> tóm tắt -> gợi ý bước tiếp theo**.

**Trả lời của bạn:**

Hãy viết 2-4 câu, trong đó có cả:

- bạn chọn lát cắt nào,
- và vì sao đây là đơn vị đủ nhỏ để eval mà vẫn chạm đúng rủi ro vận hành.

> Lát cắt tôi chọn: **một lượt hội thoại mới (tin nhắn khách + context gần đây) → AI phát hiện tín hiệu nhận diện → tra cứu CRM/OMS có điều kiện → tóm tắt + gợi ý bước tiếp theo (và nháp trả lời tùy chọn)**.
>
> Đơn vị này đủ nhỏ để eval pass/fail theo từng phiên chat, có input/output trace rõ (signals, lookup result, summary). Đủ lớn để chạm rủi ro: match nhầm khách/đơn, che ambiguity, gợi ý sai khi CRM–OMS mâu thuẫn, hoặc nháp trả lời vượt quyền.
>
> Output dùng bởi nhân viên sales/CSKH trên CRM inbox. Sai thì nhân viên trả lời sai khách, upsell sai thời điểm, hoặc lộ thông tin đơn của người khác — mất trust dù AI không tự gửi tin.

### 2. Quality Question

Viết một câu hỏi chất lượng đủ cụ thể cho lát cắt này.

Gợi ý:

- Đừng hỏi kiểu quá rộng như: “Copilot này có hữu ích không?”
- Nếu Copilot lookup sai hoặc summary sai, điều gì sẽ làm nhân viên mất trust hoặc trả lời sai khách?
- Behavior nào là bắt buộc?
- Behavior nào là bị cấm?
- Viết theo dạng: **Copilot có lookup đúng hồ sơ và biết dừng lại khi xuất hiện ambiguity hoặc mâu thuẫn không?**

**Trả lời của bạn:**

Hãy viết 2-4 câu, trong đó có cả:

- câu hỏi chất lượng bạn chọn,
- và vì sao nếu fail ở đây thì sales có thể mất trust hoặc trả lời sai khách.

> **Câu hỏi chất lượng:** Khi khách gửi tín hiệu nhận diện trong chat, Copilot có lookup đúng hồ sơ/đơn liên quan, dừng lại và cảnh báo khi ambiguous hoặc CRM–OMS mâu thuẫn, và gợi ý bước tiếp theo grounded trong dữ liệu thực — mà không tự hành động thay nhân viên?
>
> **Behavior bắt buộc:** Nhiều match → `ambiguity_warning` + không auto-chốt · Không tìm thấy → nói rõ "chưa tìm thấy" · Mâu thuẫn hệ thống → `conflict_warning` · Nháp chỉ đề xuất, không auto-send.
>
> **Behavior bị cấm:** Bịa khách/đơn khi lookup fail · Gán đơn của người khác cho khách đang chat · Tóm tắt "đã giao" khi OMS báo trạng thái khác · Tự tạo đơn hoặc gửi tin nhắn.
>
> Fail ở đây thì mock outcome (đơn "đã giao" + upsell, cảnh báo Không) có thể khiến sales báo sai trạng thái giao hàng — nhìn UI mượt nhưng judgment vận hành sai.

### 3. Output Contract tối thiểu

Không cần đoán full JSON hoàn chỉnh. Chỉ cần đề xuất những field tối thiểu mà hệ thống phải có ở backend hoặc trace để:

- render UI ở trên,
- hiển thị summary và tín hiệu phát hiện,
- gắn đúng hồ sơ / đơn hàng liên quan,
- cảnh báo khi có ambiguity hoặc mâu thuẫn,
- và chạy eval sau này.

Mẹo:

- Hãy nhìn từ khung UI gợi ý để quyết định field nào thật sự cần hiện ra.
- Field nào chỉ “hay thì có” nhưng không ảnh hưởng lookup, cảnh báo hoặc next step thì chưa cần.

**Trả lời của bạn:**

Đừng chỉ liệt kê field. Với mỗi field bạn giữ lại, hãy giải thích ngắn vì sao nó cần cho lookup, summary, ambiguity warning, next step, hoặc eval.

> | Field | Vì sao cần |
> | --- | --- |
> | `conversation_id` | Liên kết output với phiên chat — trace, eval, audit. |
> | `channel` | Enum (`zalo`, `facebook`, `web_chat`, `crm_inbox`) — context kênh, ảnh hưởng format tín hiệu. |
> | `detected_signals` | Mảng `{type, value, confidence}` — `phone`, `email`, `order_id`, `customer_id`, `product_mention` — render "Tín hiệu phát hiện" + input cho lookup. |
> | `lookup_status` | Enum (`matched`, `ambiguous`, `not_found`, `insufficient_signal`, `skipped`) — quyết định UI state và gate. |
> | `matched_customer_id` | Chỉ khi single match — null nếu ambiguous/not_found. |
> | `matched_order_id` | Chỉ khi đơn khớp và ownership consistent — null nếu conflict. |
> | `lookup_summary` | Object tóm tắt fact từ CRM/OMS (tên, trạng thái đơn, sales owner) — tách khỏi suy luận AI. |
> | `conversation_summary` | Tóm tắt khách đang hỏi gì — hiển thị Copilot, dùng LLM judge groundedness. |
> | `ambiguity_warning` | Boolean — render cảnh báo "nhiều hồ sơ khớp". |
> | `conflict_warning` | Boolean — CRM vs OMS hoặc SĐT vs order owner mâu thuẫn. |
> | `conflict_details` | String ngắn mô tả mâu thuẫn — giúp sales quyết định, không che bằng summary mượt. |
> | `suggested_next_step` | Enum + text (`ask_for_phone`, `ask_for_order_id`, `confirm_delivery`, `escalate_cskh`, `transfer_sales`, …) — gợi ý hành động nội bộ. |
> | `draft_reply` | String tùy chọn — nháp trả lời; **không** field auto-send. |
> | `confidence` | Float 0–1 — gate: thấp + ambiguous → bắt human chọn trước khi show match. |
> | `requires_human_pick` | Boolean — true khi ambiguous hoặc conflict — chặn nút "Dùng nháp" cho đến khi nhân viên xác nhận. |
>
> Không đưa vào v0: `auto_send_message`, `order_created`, `customer_updated` — AI không được tự thực hiện.

### 4. Eval Decision Map

Ở phần này, bạn phải **tự quyết định** đâu là các thành phần thật sự cần chấm.

Đừng biến bảng này thành đáp án chép lại từ đề. Hãy tự chọn các thành phần dựa trên:

- `Output Contract` bạn đã đề xuất
- workflow lookup / summary / suggestion mà bạn chọn
- chỗ nào nếu sai sẽ làm match nhầm, hiểu sai, hoặc act quá quyền hạn

| Thành phần cần chấm | Code | LLM | Human | Expert | Lý do |
| --- | ---: | ---: | ---: | ---: | --- |
| Schema + enum hợp lệ | ✓ | | | | Field bắt buộc, enum cố định, không có field auto-action. |
| Signal format (SĐT 10 số, mã đơn pattern) | ✓ | | | | Normalization deterministic — regex/parser. |
| Ambiguous lookup → không set single `matched_*_id` | ✓ | | ✓ | | Rule cứng PHI/match — human verify case biên. |
| Not found → không bịa `lookup_summary` | ✓ | ✓ | ✓ | | Code check null IDs; LLM check summary không invent facts. |
| Conflict → `conflict_warning = true` | ✓ | ✓ | ✓ | | Code: owner đơn ≠ khách match; LLM: narrative không che conflict. |
| Lookup correctness (đúng khách/đơn khi matched) | ✓ | ✓ | ✓ | | Code so ID với golden; LLM/human cho case phức tạp. |
| Conversation summary groundedness | | ✓ | ✓ | | Cần đọc hiểu hội thoại tiếng Việt, không bốc thêm ý. |
| Intent / next step appropriateness | | ✓ | ✓ | | "Đang giao" vs "đã giao" — judgment vận hành sales. |
| Draft reply safety (không hứa sai, không upsell khi conflict) | | ✓ | ✓ | | Mock outcome fail — semantic check trên nháp. |
| Khi nào nên hỏi thêm (insufficient signal) | | ✓ | ✓ | | "Case này gấp" — không có keyword lookup được. |
| Confidence calibration | | ✓ | ✓ | | Ambiguous không được confidence cao + narrative chắc chắn. |

Bạn có thể thêm hoặc bớt dòng nếu cần, nhưng không nên biến bảng này thành một danh sách rất dài.

Không chấp nhận bảng chỉ tick `Yes/No`. Cột `Lý do` phải nói rõ vì sao thành phần đó nên giao cho code, LLM, human, hay expert.

### 5. Kiểm tra tự động bằng code

Liệt kê **đầy đủ** các rule kiểm tra tự động mà bạn cho rằng case này cần có.

Không giới hạn số lượng. Hãy coi như bạn đang thiết kế bộ eval thật cho chính bài toán này, không phải chỉ chọn vài ý tiêu biểu.

Ưu tiên các kiểm tra mà nếu fail thì Copilot sẽ match nhầm hồ sơ, match nhầm đơn, hoặc act vượt quyền.

Mỗi ý nên viết theo dạng:

- Kiểm tra: [rule]
  Vì sao nên giao cho code:

- Kiểm tra: Output parse JSON và có đủ field bắt buộc (`conversation_id`, `detected_signals`, `lookup_status`, `conversation_summary`, `ambiguity_warning`, `conflict_warning`, `suggested_next_step`, `confidence`, `requires_human_pick`).
  Vì sao nên giao cho code: Invariant kỹ thuật.

- Kiểm tra: Output **không chứa** `auto_send_message`, `order_created`, `customer_updated`.
  Vì sao nên giao cho code: Action safety — AI không được tự hành động.

- Kiểm tra: `lookup_status`, `channel`, `suggested_next_step` type thuộc enum cho phép; `confidence` ∈ [0, 1].
  Vì sao nên giao cho code: Schema validation.

- Kiểm tra: SĐT trong `detected_signals` sau normalize phải 10 chữ số (VN) hoặc flag `invalid_format`.
  Vì sao nên giao cho code: Format rule trước khi gọi CRM.

- Kiểm tra: Mã đơn match pattern `DH-\d+` (hoặc taxonomy nội bộ) — sai format không được fuzzy-match bừa.
  Vì sao nên giao cho code: Tránh SC-03 (sai 1 ký tự vẫn match).

- Kiểm tra: Nếu CRM trả về >1 `customer_id` cho cùng signal thì `lookup_status = ambiguous`, `matched_customer_id = null`, `ambiguity_warning = true`.
  Vì sao nên giao cho code: Seed B — rule cứng không auto-pick.

- Kiểm tra: Nếu `lookup_status = not_found` hoặc `insufficient_signal` thì `matched_customer_id` và `matched_order_id` phải null, `lookup_summary` rỗng hoặc explicit empty.
  Vì sao nên giao cho code: Chặn bịa hồ sơ.

- Kiểm tra: Nếu `order.owner_customer_id ≠ matched_customer_id` (khi cả hai non-null) thì `conflict_warning = true`, `requires_human_pick = true`.
  Vì sao nên giao cho code: Seed C / mock — đơn không thuộc khách đang chat.

- Kiểm tra: Nếu `conflict_warning = true` thì `draft_reply` không được chứa upsell / "đã giao thành công" (keyword block list cơ bản).
  Vì sao nên giao cho code: Hard guardrail trên nháp; semantic đầy đủ giao LLM.

- Kiểm tra: Nếu `requires_human_pick = true` thì UI state phải disable auto-actions (assert trong integration test).
  Vì sao nên giao cho code: Gate vận hành trước khi sales dùng nháp.

- Kiểm tra: `detected_signals` mỗi phần tử phải có `type` + `value` + `confidence`; value phải xuất hiện trong transcript (substring hoặc normalized match).
  Vì sao nên giao cho code: Chặn hallucinate signal không có trong chat.

- Kiểm tra: `conversation_id` output khớp input.
  Vì sao nên giao cho code: Trace integrity.

- Kiểm tra (regression): Happy path DH-48291 + 0909123456 single match không được đổi `matched_order_id` sau prompt change.
  Vì sao nên giao cho code: CI regression snapshot.

### 6. Tiêu chí chấm bằng LLM

Liệt kê **đầy đủ** các tiêu chí semantic mà case này cần có và code không chấm tốt.

Không giới hạn số lượng. Hãy coi như đây là bộ tiêu chí bạn thật sự sẽ dùng để chấm case này.

Chỉ giữ những tiêu chí mà cần đọc hiểu hội thoại, tính hữu ích của summary, hoặc mức độ an toàn của gợi ý.

Mỗi ý nên viết theo dạng:

- Tiêu chí: [criterion]
  Vì sao code không bắt tốt:

- Tiêu chí: **Conversation summary accuracy** — tóm tắt có phản ánh đúng khách đang hỏi gì không?
  Vì sao code không bắt tốt: Cần đọc hiểu intent tiếng Việt, teencode, thiếu dấu.

- Tiêu chí: **Lookup narrative groundedness** — `lookup_summary` và `conversation_summary` chỉ dùng fact từ chat + lookup payload, không invent?
  Vì sao code không bắt tốt: So sánh ngữ nghĩa giữa nhiều nguồn.

- Tiêu chí: **Ambiguity handling quality** — khi nhiều hồ sơ khớp, Copilot có nói rõ uncertainty và không viết như đã chắc chắn?
  Vì sao code không bắt tốt: Tone và cách diễn đạt "có vẻ là" vs "chắc chắn là".

- Tiêu chí: **Conflict surfacing** — khi CRM–OMS mâu thuẫn, output có nêu rõ conflict thay vì gộp thành câu chuyện mượt?
  Vì sao code không bắt tốt: Mock outcome — narrative có thể pass field check nhưng che conflict.

- Tiêu chí: **Order status correctness in suggestions** — gợi ý/next step có khớp trạng thái đơn thực ("đang giao" vs "đã giao")?
  Vì sao code không bắt tốt: Cần hiểu ngữ cảnh sales và thời điểm upsell.

- Tiêu chí: **Draft reply safety** — nháp có hứa hẹn sai, tiết lộ thông tin đơn người khác, hoặc upsell khi chưa xác minh?
  Vì sao code không bắt tốt: Nhiều cách diễn đạt hứa hẹn; keyword block chỉ bắt một phần.

- Tiêu chí: **Ask-for-more appropriateness** — khi thiếu tín hiệu, `suggested_next_step` có hướng hỏi SĐT/mã đơn thay vì giả vờ đã lookup?
  Vì sao code không bắt tốt: "Case này gấp" — judgment về đủ/không đủ signal.

- Tiêu chí: **Intent boundary (sales vs CSKH vs hậu mãi)** — phân loại và gợi ý có đúng ngữ cảnh không?
  Vì sao code không bắt tốt: SC-07 — nhầm intent bán hàng thành hậu mãi.

- Tiêu chí: **Over-confidence detection** — confidence cao có đi kèm narrative chắc chắn khi lookup_status ambiguous/conflict không?
  Vì sao code không bắt tốt: Holistic read giống mock T-002 style failure.

### 7. Human / Expert Review

- Ai cần review?
- Review những case nào?
- Có cần domain expert không? Nếu không, vì sao?

**Trả lời của bạn:**

Đừng chỉ ghi tên team review. Hãy giải thích vì sao đúng nhóm đó cần xem, và họ đang kiểm tra rủi ro gì.

> **Ai review:** Team lead Sales/CSKH + 1–2 senior agent có kinh nghiệm CRM · CRM ops (người hiểu data model khách/đơn, quy tắc match).
>
> **Review case nào:**
> - 100% golden set ban đầu (~75 case) — tạo label cho LLM judge calibration.
> - Mọi `lookup_status = ambiguous` hoặc `conflict_warning = true` — trước khi nhân viên dùng nháp.
> - Mọi `confidence < 0.75` — spot-check trước auto-suggest.
> - Disagreement code pass + LLM judge fail (đặc biệt mock-style: đơn "đã giao" + upsell).
> - Tuần 1 pilot: 20% ngẫu nhiên case `matched` để calibrate.
>
> **Có cần domain expert không:** **Không.** Đây là bài toán ops/sales/CRM — taxonomy match, ambiguity, conflict đã do team vận hành và CRM ops định nghĩa. Không có nuance chuyên ngành (y khoa, pháp lý) cần expert ngoài. Human review từ người dùng Copilot hàng ngày đủ để xác nhận judgment và calibrate judge.

Nếu chọn **có domain expert**, bạn phải làm thêm 2 phần dưới đây. Nếu **không cần domain expert**, hãy ghi `Không áp dụng` và giải thích 1 câu.

#### 7A. Màn hình cho Domain Expert (ASCII)

**Trả lời của bạn:**

```text
Không áp dụng — case Sales Copilot thuộc ops/CRM;
human review từ sales lead + CRM ops là đủ (xem phần 7).
```

#### 7B. Tiêu chí review của Domain Expert

Không áp dụng.

### 8. Release Gate

Đề xuất release gate phù hợp cho case này. Nêu rõ điều kiện chặn, ngưỡng chất lượng tối thiểu, và trường hợp cần human review.

**Gate 1 — Hard block (CI, trước mọi deploy prompt/model):**

- 100% pass code rules (schema, enum, ambiguous/conflict invariants, no auto-action fields).
- 0 regression trên happy path và ambiguous baseline.
- **Chặn deploy** nếu fail rule ambiguous hoặc ownership consistency.

**Gate 2 — Quality threshold (offline eval trên golden set ~85 case):**

| Metric | Ngưỡng tối thiểu | Ghi chú |
| --- | --- | --- |
| Lookup accuracy (single match cases) | ≥ 92% | `matched_customer_id` + `matched_order_id` đúng |
| Ambiguity handling recall | ≥ 95% | Phát hiện và flag đúng khi 2+ match |
| Conflict detection recall | ≥ 93% | CRM–OMS hoặc wrong-owner |
| Summary groundedness | ≥ 90% | LLM judge |
| Draft reply safety | ≥ 95% | Không hứa sai / upsell khi conflict |
| Insufficient signal → ask more rate | ≥ 90% | Seed D class |

- **Chặn deploy** nếu ambiguity recall < 95% hoặc conflict detection < 93%.

**Gate 3 — Runtime safety (production pilot):**

- `requires_human_pick = true` → disable "Dùng nháp AI" và "Xem hồ sơ" full cho đến khi nhân viên chọn bản ghi.
- `lookup_status = insufficient_signal` → chỉ hiện gợi ý hỏi thêm, không show khách/đơn giả.
- Mọi thay đổi prompt → full offline eval trước khi > 15% traffic inbox.

**Gate 4 — Human spot-check (tuần 1 pilot):**

- 15% inbox mới: spot-check 25% output Copilot bởi sales lead.
- Scale 50% chỉ khi agreement ≥ 88% với human judgment trên 180 chat thực · 0 incident match nhầm đơn.

**Trường hợp cần human review ngay:**

- `conflict_warning = true` · Disagreement code vs LLM trên order status · Khách phản hồi "sai đơn/sai tên" trong 24h sau khi sales dùng nháp Copilot.

### 9. Kế hoạch chạy thử và dự toán chi phí

Làm phần này với giả định team của bạn vừa nhận đề bài Copilot này từ công ty.

Bạn là PM phụ trách đề xuất cách xây bộ eval, cách chạy thử, và chi phí cần xin để trả lời câu hỏi:

- hướng làm này hiện chính xác tới đâu
- có đủ an toàn và hữu ích để đem đi đề xuất tiếp hay chưa
- và với một khoản budget thử nghiệm nhỏ, team có thể chứng minh được gì

README của folder này chỉ cho khung tính. Hãy giữ cách tính đơn giản: phần người tính bằng `giờ công`, phần máy tính bằng `chi phí API key` tính từ **giá thật** của model / dịch vụ bạn chọn.

Để làm phần này, bạn cần tự tính và nêu rõ:

- giá API thật bạn dùng để tính
- tổng số cases pilot dự kiến
- tổng số lần chạy / lặp lại dự kiến
- tổng giờ PM / thiết kế eval
- tổng giờ sales ops / CRM ops
- tổng giờ human review
- nếu có `domain expert`, tổng số giờ expert
- tổng chi phí API key
- tổng chi phí pilot
- tổng thời gian dự kiến

Có thể lấy mốc tham khảo để nhẩm nhanh:

- khoảng `50-100 cases`
- khoảng `30-50 lần chạy / lặp lại`

Không cần trình bày thành bảng. Hãy tự chọn cách trình bày miễn là người đọc nhìn vào hiểu được bạn đã tính gì và chi phí tổng rơi vào đâu.

**Mục tiêu pilot:** Trả lời — (1) lookup + ambiguity handling chính xác tới đâu, (2) đủ an toàn để gợi ý nháp chưa, (3) với ~$1,350 budget chứng minh pilot 15% inbox được không.

**Giả định quy mô:**

- 85 pilot cases (4 tình huống mẫu + 5 edge + ~76 chat anonymize từ Zalo/FB)
- 40 lần chạy/lặp (4 vòng prompt × 10 full suite)
- Copilot API: 85 × 40 = **3,400 lần**
- LLM judge: 85 × 10 tiêu chí × 2 vòng = **1,700 lần**

**Giá API (OpenAI pricing, tham chiếu 06/2025 — [openai.com/api/pricing](https://openai.com/api/pricing)):**

- Copilot: **GPT-4o-mini** — $0.15/1M input, $0.60/1M output
- Judge: **GPT-4o** — $2.50/1M input, $10.00/1M output
- Mỗi lần copilot: ~900 input + 350 output tokens (hội thoại + lookup context)
- Mỗi lần judge: ~1,500 input + 120 output tokens

**Chi phí API:**

- Copilot: 3,400 × (900×0.15/1M + 350×0.60/1M) ≈ 3,400 × $0.000345 ≈ **$1.17**
- Judge: 1,700 × (1,500×2.50/1M + 120×10/1M) ≈ 1,700 × $0.00495 ≈ **$8.42**
- Buffer 20% ≈ **$1.90**
- **Tổng API ≈ $11–12**

**Giờ công người:**

| Vai trò | Giờ | Việc làm |
| --- | --- | --- |
| PM / thiết kế eval | 16h | Rubric, golden set, gates, báo cáo |
| Engineer | 14h | Eval harness, code rules, mock CRM/OMS fixture |
| Sales ops / CRM ops | 8h | Cung cấp case thật, định nghĩa ownership rules |
| Sales lead + senior CSKH | 12h | Label 85 case, review ambiguity/conflict disagreement |
| **Tổng giờ người** | **50h** | |

- Chi phí nội bộ: **$30/giờ** blended
- **Chi phí người: 50h × $30 = $1,500**

**Tổng chi phí pilot: ~$1,510** (API ~1%, nhân công ~99%)

**Timeline: 3 tuần**

- Tuần 1: Thu thập 85 chat, human label golden set, mock CRM/OMS fixtures
- Tuần 2: 2 vòng prompt + code rules + LLM judge calibration
- Tuần 3: 2 vòng iteration, chốt gates, pilot 15% inbox + spot-check

**Pilot chứng minh được gì:**

- Con số lookup accuracy và ambiguity recall trên chat tiếng Việt thật
- Copilot không match nhầm đơn wrong-owner (Seed C class)
- Code rules + judge chạy CI trước deploy
- Chi phí API không đáng kể — rủi ro chính là match sai và narrative mượt che conflict

**Tóm tắt:**

Giá API từ OpenAI pricing (GPT-4o-mini + GPT-4o). Với 85 case và 40 lần chạy, tổng **~$1,510**, API chỉ ~$12. Plan đủ nhỏ để xin budget, đủ cover failure mode nguy hiểm nhất (ambiguous match, conflict che bởi summary mượt) để quyết định pilot 15% inbox.

---
