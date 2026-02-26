# AI Showcase — Test Case Generator
> Tài liệu này tổng hợp các prompt/conversation AI tốt nhất để trình bày với BGK khi present.

## 1) Giới thiệu

- **User mục tiêu:** QC/Manual Tester, QA Lead.
- **Problem:** Viết test case thủ công tốn nhiều thời gian, dễ thiếu edge case/security case, format không đồng nhất.
- **Business value:** Rút ngắn thời gian viết TC từ nhiều giờ xuống còn 15–30 phút, tăng coverage, chuẩn hoá output toàn team.
- **Skill demo:** Nhận spec `.md` → sinh bộ test case đầy đủ (Happy Path/Negative/Edge/Security) + Traceability Matrix + Coverage Summary.

---

## 2) Checklist “đủ điều kiện show BGK” 

- [x] Có prompt thực tế bám theo spec thật.
- [x] Có conversation nhiều vòng (không chỉ 1 prompt).
- [x] Có output cụ thể, dùng được ngay cho test execution.
- [x] Có minh hoạ khả năng refine kết quả bằng prompt follow-up.
- [x] Có phần nêu hạn chế và cách xử lý khi spec thiếu dữ liệu.
- [ ] Đã gắn **screenshot thật** từ phiên chat khi present.
- [ ] Đã gắn **link share conversation** (Claude/ChatGPT) bản cuối.

---

## 3) Showcase Prompt #1 — Generate full test suite from spec

<img width="778" height="756" alt="image" src="https://github.com/user-attachments/assets/4dff3694-b482-44d8-b16f-c580f1891e4d" />

## 4) Showcase Prompt #2 — Refine output để thêm edge/security chiều sâu

### Prompt follow-up

Refine bộ test case ở trên với yêu cầu:
- Bổ sung boundary value cho các field có min/max.
- Bổ sung case về whitespace, null, format sai.
- Bổ sung security case: SQL injection, brute force, rate limit, auth bypass.
- Nếu test case nào chưa rõ expected result thì viết lại expected result đo được/kiểm chứng được.
- Đánh dấu [NEW] cho các case mới được thêm.

<img width="867" height="1078" alt="image" src="https://github.com/user-attachments/assets/5965f6ec-aa4e-4858-b381-8859a5fd870a" />

## 5) Showcase Prompt #3 — Build skill từ file spec đã được duyệt

```text
Build file skill dựa vào file spec.md...
```

- Tạo `SKILL.md` theo đúng format chuẩn skill.
- Bao gồm: persona, input format, workflow, output schema, checklist đánh giá coverage.
- Nếu spec thiếu thông tin, thêm bước hỏi lại để làm rõ trước khi sinh test case.
- Ưu tiên output có thể copy dùng ngay cho test execution.

## 6) Showcase Prompt #4 — Refine skill sau lần đầu chạy thử

<img width="833" height="1055" alt="image" src="https://github.com/user-attachments/assets/c5d189a6-fe76-476a-9698-652e41552edc" />

## 7) Showcase Prompt #5 — Build và refine file spec từ 1 project thực tế

<img width="781" height="1065" alt="image" src="https://github.com/user-attachments/assets/ec6b804a-7f42-4eed-9bc1-4e288053e25a" />

<img width="783" height="1035" alt="image" src="https://github.com/user-attachments/assets/2766a863-f4e0-4e63-84ae-f229bf36d52b" />

## 8) Showcase Prompt #6 — Run file skill.md từ spec thực tế và đánh giá độ coverage của output

<img width="2541" height="1347" alt="image" src="https://github.com/user-attachments/assets/ba15c813-5e62-4490-b53e-07e8fffdf7a3" />

<img width="736" height="420" alt="image" src="https://github.com/user-attachments/assets/930b6bb9-306a-4fa9-bd62-27429df46060" />

## 9) Showcase Prompt #7 — Tiếp tục refine skill

<img width="779" height="1083" alt="image" src="https://github.com/user-attachments/assets/256010be-c848-4639-8629-d1674e114668" />

## 10) Showcase Prompt #8 — Run file skill từ spec thực tế và đánh giá độ coverage của output

<img width="878" height="569" alt="image" src="https://github.com/user-attachments/assets/c5b2590c-317b-48bc-b4e9-d8a73045f607" />

<img width="746" height="1065" alt="image" src="https://github.com/user-attachments/assets/87392f08-9b94-46cd-afa9-0e51de6ab763" /> 
