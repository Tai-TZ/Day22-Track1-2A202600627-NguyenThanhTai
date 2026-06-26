# Ngày 22 - Eval Decision Workbook

## Nộp bài

Mỗi học viên nộp **1 repository riêng**.

Tên repo phải theo đúng format:

```text
Day22-Track1-MãHV-HọVàTên
```

## Vai trò của bạn

Bạn là PM phụ trách đề xuất cách xây bộ eval, cách chạy thử, và chi phí cần xin để trả lời 3 câu hỏi:

- hướng làm này hiện chính xác tới đâu
- có đủ an toàn và hữu ích để đem đi đề xuất tiếp hay chưa
- với một khoản budget thử nghiệm nhỏ, team có thể chứng minh được gì

## Bài này để làm gì?

Ngày 22 không phải bài build full system hay viết eval runner. Mục tiêu của workbook này là luyện **phản xạ ra quyết định thiết kế eval** khi đứng trước một bài toán AI thật.

Bạn sẽ luyện 3 kỹ năng chính:

- xác định `Unit of Work` đủ nhỏ để đánh giá
- chọn đúng cách chấm: `code`, `LLM`, `human`, hay `domain expert`
- nghĩ như một người PM đang nhận đề bài thật từ công ty: cần lên chiến lược eval, kế hoạch chạy thử, và chi phí để chứng minh hướng làm có thể chạy được

Vì sao cần:

- nhiều hệ thống AI không fail vì model trước, mà fail vì không rõ đang đo cái gì
- không rõ ai là người có quyền xác nhận chất lượng
- và không có release gate đủ chặt trước khi đưa vào vận hành

Workbook này giúp bạn tập đúng chỗ đó: từ bài toán AI, đi tới quyết định đánh giá, release gate, và pilot plan ở mức sơ bộ.

## Mỗi case sẽ đi qua những phần gì?

Mỗi case đều xoay quanh một flow giống nhau:

- xác định `Unit of Work` và `Quality Question`
- đề xuất `Output Contract` tối thiểu
- chọn cách chấm trong `Eval Decision Map`
- đặt `Release Gate` và nghĩ ra `Dataset Edge Cases`
- sketch workflow / UI bằng ASCII khi cần
- lập một `pilot plan` ngắn để ước lượng nguồn lực

Nếu case có `domain expert`, bài làm phải có thêm:

- màn hình review cho expert bằng ASCII
- tiêu chí để expert dùng khi duyệt

## Không cần làm

- Không cần full reference dataset lớn.
- Không cần eval runner, prompt implementation, hay code chạy thật.
- Không cần làm dự toán tài chính quá chi tiết.
- Không cần mockup ngoài ASCII.

## Nguyên tắc chung

- Mọi quyết định quan trọng phải có **lý do**.
- UI, workflow, và màn hình cho expert đều phải sketch bằng **ASCII**.
- Nếu chọn `code`, phải nói vì sao phần đó đủ deterministic.
- Nếu chọn `LLM`, phải nói vì sao code không bắt tốt.
- Nếu chọn `human` hoặc `expert`, phải nói rõ họ đang xác nhận điều gì.
- Nếu đề xuất `edge case`, phải nói case đó dùng để bắt failure gì.
- Nếu đề xuất `budget` hoặc `timeline`, phải ghi rõ bộ giả định tạo ra con số đó.

## PM Plan, Budget, và Timeline

Phần này không phải bài estimate chi tiết. Bạn chỉ cần ước lượng đủ để ra quyết định và đủ để đem đi xin budget chạy thử.

Khi làm phần này, hãy tự tính và nêu rõ:

- giá API thật bạn dùng để tính
- tổng số cases pilot dự kiến
- tổng số lần chạy / lặp lại dự kiến
- các vai trò nào cần tham gia và số giờ công của từng vai trò
- nếu có `domain expert`, expert cần bao nhiêu giờ và để xác nhận điều gì
- tổng chi phí pilot và tổng thời gian dự kiến

Có thể lấy mốc tham khảo để nhẩm nhanh:

- khoảng `50-100 cases`
- khoảng `30-50 lần chạy / lặp lại`

Không cần trình bày thành bảng. Mục tiêu là ra được một con số sơ bộ đủ để trả lời: với một khoản budget thử nghiệm nhỏ, team sẽ chứng minh được gì.

## Human Review và Domain Expert

Trong mọi case, học viên phải chỉ ra rõ:

- phần nào giao cho code,
- phần nào cần human review,
- phần nào cần domain expert.

Nếu **không cần** domain expert, vẫn phải giải thích vì sao human review vận hành là đủ.

Nếu **cần** domain expert, câu trả lời chưa đủ nếu thiếu 2 phần:

- một màn hình review cho expert bằng ASCII
- một bộ tiêu chí để expert dùng khi duyệt

Expert thường được dùng để xác nhận:

- taxonomy hoặc policy của domain
- rubric cho case khó
- edge case high-risk
- release gate trước khi ship
- failure nghiêm trọng cần nhìn dưới góc độ chuyên môn

Màn hình review cho expert nên cho thấy tối thiểu:

- AI đã kết luận gì
- expert cần nhìn lại dữ liệu hoặc evidence nào
- expert có thể duyệt / bác bỏ / sửa / escalate ở đâu

## 3 Case và brief nhanh

| File | Bài toán chính | Mức scaffold | Cách bắt đầu gợi ý |
| --- | --- | --- | --- |
| `01-ticket-triage.md` | AI đọc ticket hỗ trợ và gợi ý phân loại, mức độ khẩn, route, escalation | Cao | Đọc ví dụ full luồng, nhìn từ UI để suy ra output contract, rồi điền decision map |
| `02-sales-copilot.md` | AI đọc chat với khách, phát hiện tín hiệu tra cứu, lookup nội bộ, tóm tắt và gợi ý bước tiếp theo | Trung bình | Dùng workflow tham khảo làm nháp, chốt ranh giới giữa lookup, hỏi thêm, và gợi ý |
| `03-medical-routing.md` | AI tóm tắt cuộc gọi y tế, phát hiện red flag, lookup hồ sơ, và route sang đúng người hoặc đúng quy trình | Thấp | Viết `Unit of Work` và `Quality Question` trước, rồi mới thiết kế workflow, UI, checkpoint human/expert |

Khi cần kiểm tra lại hướng làm, hãy quay về câu hỏi:

> Hệ thống đang phải ra quyết định gì, quyết định đó hiển thị ra đâu, và sai ở bước nào thì hậu quả lớn nhất?

Tài liệu tham khảo:

- `./ai-evals-reference-guide-vi.md`

---

## Bài nộp — Nguyen Thanh Tai

**Mã học viên:** 2A202600627  
**Repository:** `Day22-Track1-2A202600627-NguyenThanhTai`  
**Trạng thái:** Đã hoàn thành workbook cho cả 3 case + presentation HTML cho stakeholder.

### Cấu trúc repository

```text
Day22-Track1-2A202600627-NguyenThanhTai/
├── README.md
├── ai-evals-reference-guide-vi.md      # Tài liệu tham khảo eval
├── 01-ticket-triage.md                 # Case 1 — workbook (đã điền)
├── 02-sales-copilot.md                 # Case 2 — workbook (đã điền)
├── 03-medical-routing.md               # Case 3 — workbook (đã điền)
├── case-1/
│   └── 01-ticket-triage-presentation.html
├── case-2/
│   └── 02-sales-copilot-presentation.html
└── case-3/
    └── 03-medical-routing-presentation.html
```

Mở file `.html` trong trình duyệt để xem deck trình bày (scroll hoặc dùng nav dots).

### Tóm tắt 3 case

| Case | Bài toán | Domain expert | Pilot (ước lượng) | Gate ưu tiên |
| --- | --- | --- | --- | --- |
| **1** Ticket Triage | AI phân loại ticket, route, escalation | Không — human review từ support lead đủ | ~$1,210 · 3 tuần · 80 cases | Escalation recall ≥ 95%, under-triage ≤ 2% |
| **2** Sales Copilot | Detect signal → lookup CRM/OMS → summary + nháp | Không — sales lead + CRM ops đủ | ~$1,510 · 3 tuần · 85 cases | Ambiguity recall ≥ 95%, conflict detection ≥ 93% |
| **3** Medical Routing | Tóm tắt cuộc gọi, red flag, route y tế | **Có** — ĐD trưởng + BS trực (24h) | ~$2,820 · 4 tuần · 90 cases | Red flag recall ≥ 98%, diagnostic overreach 0% |

### Điểm thiết kế chung

- **Ba tầng chấm:** code (CI invariant) → LLM judge (semantic, calibrated) → human/expert (golden label + case biên).
- **Mock failure làm mốc eval:** Case 1 (T-002 under-triage), Case 2 (lookup mượt nhưng sai đơn/conflict), Case 3 (bỏ sót red flag, route CSKH).
- **Chi phí API không đáng kể** ở quy mô pilot (~$10–14/case); phần lớn budget là giờ label/review (Case 3 thêm giờ expert).
- **Giá API tham chiếu:** [OpenAI Pricing](https://openai.com/api/pricing) — GPT-4o-mini (triage/copilot), GPT-4o (LLM judge).

### Cách xem nhanh từng case

| Case | Workbook | Presentation |
| --- | --- | --- |
| 1 — Support Ticket Triage | [01-ticket-triage.md](./01-ticket-triage.md) | [case-1/01-ticket-triage-presentation.html](./case-1/01-ticket-triage-presentation.html) |
| 2 — Sales Chat Copilot | [02-sales-copilot.md](./02-sales-copilot.md) | [case-2/02-sales-copilot-presentation.html](./case-2/02-sales-copilot-presentation.html) |
| 3 — Medical Call Routing | [03-medical-routing.md](./03-medical-routing.md) | [case-3/03-medical-routing-presentation.html](./case-3/03-medical-routing-presentation.html) |

### Câu hỏi PM — câu trả lời tổng hợp

1. **Hướng làm chính xác tới đâu?**  
   Mỗi case có golden set 80–90 case, code rules + LLM judge với ngưỡng cụ thể (lookup/route/red flag tùy domain). Chưa ship production — pilot offline + spot-check trước scale.

2. **Đủ an toàn để đề xuất tiếp chưa?**  
   Case 1–2: đủ để pilot 10–15% traffic với human spot-check nếu Gate 1+2 pass. Case 3: cần expert ký Gate 2 và 100% review nhánh medical trong tuần đầu pilot.

3. **Budget nhỏ chứng minh được gì?**  
   Số liệu accuracy/recall trên case thật (anonymize), bộ rule chạy CI, và quy trình Go/No-Go — không chỉ demo "trông ổn" trên mock outcome.

