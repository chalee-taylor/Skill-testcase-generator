# Skill Card 

## testcase-generator (v1.1.0)

| Mục | Nội dung |
|---|---|
| Tên Skill | `testcase-generator` |
| Việc gì được automate | **Đầu vào:** `spec_content` (PRD/BRS/User Story).<br><br>**Xử lý tự động:** validate spec, parse flow/rule, quét rủi ro, sinh test case theo coverage (`basic/standard/full`).<br><br>**Đầu ra:** Happy Path + Negative + Edge Case + Security, kèm Intelligence Scan Report, Traceability Matrix, hỗ trợ `markdown/json/csv`. |
| TRƯỚC: làm tay | 1) QC đọc spec thủ công.<br>2) Tự phân tích flow/rule.<br>3) Viết test case thủ công.<br>4) Tự rà soát thiếu case và format.<br><br>**Thời gian:** thường **2–4 giờ/feature** (có thể **6–8 giờ** nếu phức tạp). |
| SAU: có skill | 1) Nhập spec vào skill.<br>2) Skill tự validate + phân tích + sinh test case chuẩn.<br>3) QC chỉ review/chỉnh nhẹ trước khi dùng.<br><br>**Thời gian:** khoảng **15–30 phút/feature**. |
| Tool/AI đã dùng | - **Claude/Claude Code:** phân tích spec và sinh test case.<br>- **Markdown/JSON/CSV:** định dạng output.<br>- **Git/GitHub:** quản lý version và review. |
| Limitation | 1) Spec thiếu chất lượng (không có flow hoặc business rule/validation) thì dừng với `WARN-001`.<br>2) Chỉ generate test case, không execute test.<br>3) Chưa push trực tiếp Jira/TestRail mặc định.<br>4) Spec nhiều feature nên tách nhỏ để kết quả rõ hơn. |
| Roadmap mở rộng | 1) Detect phần mơ hồ trong spec và gợi ý cải thiện trước khi generate.<br>2) Export/integration Jira-TestRail qua API.<br>3) Sinh automation skeleton từ test case (Playwright/Pytest).<br>4) Batch processing nhiều feature.<br>5) Tối ưu template đa ngôn ngữ. |
