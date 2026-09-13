# 📊 BÁO CÁO THU HOẠCH NGHIỆM THU BÀI LAB 3 (BƯỚC 3 — SUBMISSION ARTIFACT)

> **Họ và Tên Học viên:** Nguyễn Lê Ngọc Bảo  
> **Mã Sinh Viên / Mã Học viên:** 2A202602852  
> **Chủ đề Lựa chọn:** Gợi ý 1.1 — Trợ lý Học vụ & Tra cứu Lịch thi VinUni (tra cứu hồ sơ học vụ + đặt lịch tư vấn với Cố vấn học tập)  

---

## 1. BẢNG CHẤM ĐIỂM AGENTIC FIT SCORING MATRIX (ĐÁNH GIÁ CHỦ ĐỀ)

| Tiêu chí Đánh giá | Mức độ (1 - 5) | Giải trình chi tiết lý do chọn điểm |
| :--- | :---: | :--- |
| **1. Multi-step Reasoning** | **4** / 5 | Nhiều yêu cầu phải chia thành chuỗi bước nối tiếp. Ví dụ TC04: "đặt lịch với cố vấn của tôi" → (1) tra cứu hồ sơ SV để biết cố vấn là ai → (2) trích tên cố vấn + chuẩn hóa thời gian → (3) gọi tool đặt lịch → (4) xác nhận cho SV. Không chấm 5 vì chuỗi chỉ dài 2–3 bước, không cần lập kế hoạch phức tạp. |
| **2. Tool Interaction** | **5** / 5 | Mọi thông tin cá nhân (GPA, email, trạng thái, cố vấn) nằm trong CSDL học vụ bên ngoài, LLM không thể tự biết. Việc đặt lịch là **hành động ghi** (tạo booking) nên bắt buộc đi qua tool trên MCP Server (`academic_query`, `schedule_appointment`). Thiếu tool thì Chatbot chỉ có thể từ chối hoặc bịa dữ liệu. |
| **3. Dynamic Decision** | **4** / 5 | Bước sau phụ thuộc Observation bước trước: nếu `academic_query` trả `SUCCESS` → lấy trường `advisor` để đặt lịch; nếu `NOT_FOUND` (TC05) → dừng, báo lỗi lịch sự, không đặt lịch. Agent cũng phải tự quyết có cần gọi tool hay trả lời thẳng (TC01). Không chấm 5 vì số nhánh rẽ còn ít (2 tool, vài trạng thái). |
| **4. Long Horizon Goal** | **3** / 5 | Trong một phiên, Agent phải giữ mục tiêu gốc "đặt được lịch hẹn" xuyên suốt các vòng Thought → Action → Observation (không dừng ở bước tra cứu). Tuy nhiên nhiệm vụ kết thúc trong vài lượt, không kéo dài nhiều ngày/nhiều phiên và chưa cần Memory dài hạn nên chỉ ở mức trung bình. |
| **TỔNG ĐIỂM AGENTIC FIT** | **16 / 20** | *Tổng 16 > 12/20 → Bài toán **phù hợp** triển khai ReAct Agent (Cấp 3). Riêng câu hỏi quy chế chung (TC01) vẫn trả lời trực tiếp, không gọi tool để tiết kiệm token & độ trễ.* |

---

## 2. TRÍCH XUẤT KẾT QUẢ WATERFALL TRACE LOG (SAU KHI CHẠY TEST SUITE TRÊN API THẬT)

> ⚠️ **YÊU CẦU NGHIỆM THU:** Mở tệp `.env` điền `GEMINI_API_KEY` (hoặc `OPENAI_API_KEY`) để kết nối LLM thật trước khi thực thi `python src/app.py --all`. Bài nộp chỉ dùng Mock Offline Provider sẽ không đạt điểm nghiệm thực tế.

**Cấu hình chạy nghiệm thu:** `LLM_PROVIDER=gemini`, `LLM_MODEL=gemini-3.5-flash-lite` (Google GenAI SDK, Native Function Calling) — lệnh `python src/app.py --all`, kết quả **10 sự kiện trace, 0 lần fallback về Mock**.

Dán 1 đoạn trích xuất log tiêu biểu từ file `docs/trace_waterfall.json` sinh ra từ phản hồi LLM API thật — **TC04 (multi-step reasoning)**: Agent tự tra cứu cố vấn trước, dùng Observation `advisor` làm tham số để đặt lịch, rồi mới trả lời:

```json
[
  {
    "step": 1,
    "query": "Mình là sinh viên SV2026002, mình không nhớ cố vấn học tập của mình là ai. Bạn tìm giúp rồi đặt lịch hẹn tư vấn với cố vấn đó vào 09:30 ngày 18/09/2026 nhé.",
    "action_type": "TOOL_EXECUTION",
    "thought": "Gemini quyết định gọi công cụ 'academic_query' với tham số: {\"student_id\": \"SV2026002\"}",
    "tool_name": "academic_query",
    "arguments": { "student_id": "SV2026002" },
    "observation": {
      "status": "SUCCESS",
      "student_id": "SV2026002",
      "data": {
        "full_name": "Trần Thị Bình",
        "class": "AI-K4",
        "gpa": 3.6,
        "email": "binh.tt@vinuni.edu.vn",
        "status": "Đang học",
        "advisor": "TS. Lê Thị B"
      }
    },
    "latency_ms": 32408.95
  },
  {
    "step": 2,
    "action_type": "TOOL_EXECUTION",
    "thought": "Gemini quyết định gọi công cụ 'schedule_appointment' với tham số: {\"student_id\": \"SV2026002\", \"datetime_str\": \"09:30 18/09/2026\", \"advisor_name\": \"TS. Lê Thị B\"}",
    "tool_name": "schedule_appointment",
    "arguments": {
      "student_id": "SV2026002",
      "datetime_str": "09:30 18/09/2026",
      "advisor_name": "TS. Lê Thị B"
    },
    "observation": {
      "status": "SUCCESS",
      "booking_id": "BK-SV2026002-99",
      "student_id": "SV2026002",
      "datetime": "09:30 18/09/2026",
      "advisor": "TS. Lê Thị B",
      "message": "Đặt lịch thành công cho sinh viên SV2026002 với TS. Lê Thị B vào lúc 09:30 18/09/2026."
    },
    "latency_ms": 1993.64
  },
  {
    "step": 3,
    "action_type": "FINAL_ANSWER",
    "thought": "Gemini phản hồi trực tiếp bằng văn bản (không cần gọi công cụ).",
    "output": "Chào bạn Trần Thị Bình, Dựa vào thông tin học vụ của bạn, Cố vấn học tập của bạn là **TS. Lê Thị B**. Tôi đã hỗ trợ bạn đặt lịch hẹn tư vấn học vụ thành công với cố vấn. [...] **Thời gian:** 09:30 ngày 18/09/2026 - **Mã đặt lịch (Booking ID):** BK-SV2026002-99 [...]",
    "latency_ms": 2536.57
  }
]
```
*(Trường `query` của step 2–3 và phần giữa `output` được lược bớt `[...]` cho gọn; bản đầy đủ nằm trong `docs/trace_waterfall.json`.)*

### Tóm tắt Waterfall 5 Test Cases (LLM API thật)

| TC | Loại | Chuỗi thực thi (Thought → Action → Observation → Final) | Kết quả | Latency các bước (ms) |
| :---: | :--- | :--- | :---: | :--- |
| TC01 | direct_query | FINAL_ANSWER (không gọi Tool) | ✅ Đúng kỳ vọng | 3092.73 |
| TC02 | single_tool_query | `academic_query(SV2026001)` → SUCCESS → FINAL_ANSWER | ✅ | 31956.13 → 1731.19 |
| TC03 | appointment_booking | `schedule_appointment(SV2026001, 14:00 15/09/2026, PGS.TS Nguyễn Văn A)` → SUCCESS → FINAL_ANSWER | ✅ | 1638.58 → 93563.48 |
| TC04 | multi_step_reasoning | `academic_query(SV2026002)` → advisor = TS. Lê Thị B → `schedule_appointment(...)` → SUCCESS → FINAL_ANSWER | ✅ 3 bước | 32408.95 → 1993.64 → 2536.57 |
| TC05 | edge_case_handling | `academic_query(SV9999999)` → NOT_FOUND → FINAL_ANSWER xin lỗi, không bịa dữ liệu, không đặt lịch | ✅ | 62883.12 → 97847.58 |

**Nhận xét quan sát (Observability):**
- Các bước có latency ~1.6–3.1 s là thời gian gọi Gemini thực tế. Các bước ~32 s / ~63 s / ~94 s / ~98 s **bao gồm thời gian chờ retry** do Gemini Free Tier trả lỗi `429 RESOURCE_EXHAUSTED` (giới hạn request/phút); provider tự chờ rồi gọi lại thay vì fallback về Mock — trace log giúp phát hiện ngay điểm nghẽn này.
- Vòng lặp ReAct thật sự nạp Observation trở lại cho LLM: ở TC04, tham số `advisor_name` của bước 2 được lấy từ Observation bước 1 (người dùng không cung cấp).
- Hạn chế nhỏ: ở TC05, `output` của Gemini còn lẫn tiền tố "Thought: ..." trong văn bản trả lời cuối — có thể cải thiện bằng prompt yêu cầu chỉ trả về câu trả lời cho sinh viên.

---

## 3. TỔNG KẾT KẾT QUẢ NGHIỆM THU & NỘP BÀI

- [x] Đã điền API Key thật trong `.env` và xác nhận Agent chạy mượt mà trên LLM API thật (Gemini/OpenAI).
- **Tổng số Test Cases đã chạy thành công:** 5 / 5 test cases.
- **Số lượt gọi Tool qua MCP Server chính xác:** 5 lượt (TC02: 1, TC03: 1, TC04: 2, TC05: 1; TC01 đúng kỳ vọng không gọi Tool).
- **Kết quả đẩy Repo nộp bài:** [ ] Đã Commit và Push mã nguồn thành công lên GitHub cá nhân.

---

> ✅ **HOÀN TẤT NỘP BÀI:** Sao chép đường link GitHub Repository cá nhân của bạn và dán vào ô nộp bài trên hệ thống LMS VLearn để hoàn tất Bài Lab 3!
