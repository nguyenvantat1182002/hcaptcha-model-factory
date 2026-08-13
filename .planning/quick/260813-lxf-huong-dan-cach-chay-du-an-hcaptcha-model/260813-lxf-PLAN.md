---
id: 260813-lxf
slug: huong-dan-cach-chay-du-an-hcaptcha-model
date: 2026-08-13
status: in-progress
---

# Quick Plan: Hướng dẫn cách chạy dự án hcaptcha-model-factory

## Task Objective
Khảo sát và tổng hợp hướng dẫn chi tiết cách cài đặt môi trường, vận hành các tính năng chính của hcaptcha-model-factory (CLI Scaffold, Auto-labeling, Training, ONNX Export, Frontend) và cung cấp câu trả lời rõ ràng cho người dùng.

## Tasks

### Task 1: Khảo sát và tổng hợp tài liệu vận hành
- **Files:** `src/apis/scaffold.py`, `src/main.py`, `automation/_04_mini_workflow.py`, `hcaptcha-model-factory.wiki/Home.md`
- **Action:** Trích xuất luồng xử lý, tham số CLI và các bước tương tác từ mã nguồn và Wiki.
- **Verify:** Đảm bảo tất cả các câu lệnh CLI (`new`, `train`, `val`, `test_onnx`, `trainval`) hoạt động chính xác theo đúng cú pháp.
- **Done:** Tài liệu `260813-lxf-RESEARCH.md` được khởi tạo và xác minh đầy đủ thông tin.

### Task 2: Trả lời người dùng bằng Tiếng Việt
- **Action:** Cung cấp hướng dẫn sử dụng chi tiết, trực quan và dễ hiểu theo đúng ngôn ngữ yêu cầu.
