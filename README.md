# 🧠 AI Skill – Test Case Generator

## 1. Project Overview

**Skill Test Case Generator** là một dự án nhằm xây dựng AI Skill hỗ trợ QC/Tester tự động tạo test case từ file spec sản phẩm.

Mục tiêu của dự án là:

* Giảm thời gian viết test case thủ công
* Tăng độ coverage (happy path, edge case, negative case)
* Chuẩn hóa format test case giữa các QC
* Giúp QC tập trung vào review thay vì viết từ đầu

Dự án này được xây dựng trong bối cảnh cuộc thi **Cook A Skill – QC Team**, nơi mỗi cá nhân tạo một AI Skill phục vụ workflow QC thực tế.

---

## 2. Problem Statement

Trong quy trình QC hiện tại:

* QC phải đọc spec thủ công
* Tự suy nghĩ test case từng bước
* Dễ miss edge case
* Mỗi người viết format khác nhau
* Tốn nhiều thời gian cho module phức tạp

➡️ Cần một AI Skill có thể:

* Đọc spec .md
* Phân tích logic
* Generate test case chuẩn format
* Gợi ý edge case và test data

---

## 3. Target Users

* QC / Tester Manual
* QA Lead cần review test case
* Product team muốn kiểm tra coverage spec

Use case chính:

* Viết test case nhanh cho feature mới
* Review spec để tìm missing logic
* Chuẩn hóa test case trước khi test

---

## 4. Expected Workflow

Luồng hoạt động mong muốn:

1. QC cung cấp file spec của feature (markdown)
2. AI đọc spec theo instruction của Skill
3. AI phân tích:

   * User flow
   * Business logic
   * Validation rules
   * Edge cases
4. AI generate danh sách test cases
5. QC review và chỉnh sửa nếu cần

Output phải dùng được ngay cho test.

---

## 5. Input & Output

### Input

* File spec feature (.md)
* Hoặc nội dung feature description

Spec có thể ở dạng:

* PRD
* BRS
* Feature description
* User story + acceptance criteria

### Output

Danh sách test cases gồm:

* ID
* Title
* Precondition
* Steps
* Expected Result
* Priority
* Test Type (Happy / Negative / Edge)

Có thể export sang Markdown / CSV / Excel.

---

## 6. Scope of Skill

### In Scope

* Generate functional test cases
* Detect edge cases cơ bản
* Generate test data suggestion
* Chuẩn format output

### Out Scope

* Automation script
* Performance testing script
* Security penetration testing
* Integration với tool bug tracking

---

## 7. Quality Expectations

Skill cần đảm bảo:

* Coverage đầy đủ:

  * Happy path
  * Negative case
  * Edge case
* Test case rõ ràng, từng bước
* Expected result cụ thể
* Priority hợp lý
* Không duplicate test case

---

## 8. Assumptions

* Spec được viết tương đối rõ ràng
* QC có thể chỉnh sửa spec trước khi generate
* AI không thay thế QC review cuối cùng

---

## 9. Limitations

Skill có thể gặp hạn chế khi:

* Spec quá mơ hồ
* Thiếu business rule
* Không có user flow
* Feature quá phức tạp hoặc phụ thuộc nhiều hệ thống

Trong các trường hợp này, QC cần refine spec trước.

---

## 10. Next Steps

Từ README này, Agent sẽ:

1. Hiểu mục tiêu dự án
2. Viết **spec.md** cho từng feature
3. Viết **SKILL.md** để hướng dẫn AI generate test case

Repo structure dự kiến:

```
repo/
  ├── README.md
  ├── spec.md
  ├── SKILL.md
  ├── skill-card.md
  └── ai-showcase/
```

---

## 11. Success Criteria

Skill được xem là thành công khi:

* Feed 1 spec thật → generate test case usable
* QC chỉ cần chỉnh sửa nhỏ
* Coverage tốt hơn manual
* Format consistent
* Demo live chạy ổn định

---

## 12. Vision

Trong tương lai, Skill có thể mở rộng:

* Detect ambiguity trong spec
* Generate automation test skeleton
* Tạo test report tự động
* Tích hợp Jira / TestRail
* Multi‑language spec parsing
