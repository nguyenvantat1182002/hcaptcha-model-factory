# Research Findings: Tính khả thi nâng cấp tự động tạo & huấn luyện mô hình cho `image_label_area_select` và `image_drag_drop`

**Task ID:** 260813-m1h  
**Date:** 2026-08-13  
**Target:** Khảo sát khả năng mở rộng kiến trúc hcaptcha-model-factory để hỗ trợ 2 dạng bài toán hCaptcha nâng cao: Area Select và Drag-Drop.

---

## 1. Khảo sát hiện trạng codebase (Current Baseline)

Qua phân tích mã nguồn hiện tại:
1. **Đã khai báo Hằng số TaskType ([src/factories/kernel.py](file:///d:/Python/hcaptcha-model-factory/src/factories/kernel.py)):**
   ```python
   class TaskType:
       IMAGE_LABEL_BINARY = "image_label_binary"          # Đã hoàn thiện (ResNetMini)
       IMAGE_LABEL_AREA_SELECT = "image_label_area_select"# Đã khai báo stub
       IMAGE_LABEL_MULTIPLE_CHOICE = "image_label_multiple_choice"
   ```
2. **Đã có sẵn bộ sinh dữ liệu tổng hợp (Synthetic Dataset Generator):**
   - Trong [src/components/dataset_gen/on_select.py](file:///d:/Python/hcaptcha-model-factory/src/components/dataset_gen/on_select.py), [on_select_animal.py](file:///d:/Python/hcaptcha-model-factory/src/components/dataset_gen/on_select_animal.py), đã có sẵn lớp `OnSelectDatasetGen` dùng để ghép các ảnh đối tượng lên ảnh nền và tự động tạo nhãn YOLO Bounding Box (`labels/*.txt`).
3. **Đã có sẵn mô hình đánh giá YOLOv8 ([evaluation/eva_yolo_det_model.py](file:///d:/Python/hcaptcha-model-factory/evaluation/eva_yolo_det_model.py)):**
   - Đã tích hợp `hcaptcha_challenger.YOLOv8` để thực thi suy luận Object Detection từ mô hình `.onnx`.

---

## 2. Phân tích giải pháp nâng cấp chi tiết

### A. Dạng 1: `image_label_area_select` (Chọn vùng / Khoanh vùng đối tượng)

- **Mô tả bài toán:** hCaptcha yêu cầu khoanh vùng / nhấp vào vị trí đối tượng (ví dụ: khoanh vùng con bướm, chiếc xe, con vật nhỏ nhất).
- **Mô hình mục tiêu:** **YOLOv8 / YOLOv11 (Detection / Segmentation)** hoặc **Zero-Shot Object Detection (YOLO-World / Grounding DINO)**.
- **Giải pháp kiến trúc đề xuất:**
  1. *Thêm `YOLOFactory` kế thừa `ModelFactory`:* Tạo `src/factories/yolo.py` song song với `src/factories/resnet.py`.
  2. *Auto-Labeling không cần con người:* Tích hợp `YOLO-World` hoặc `Grounding DINO` vào `src/components/auto_label/yolo_labeler.py` để tự động phát hiện đối tượng từ prompt tiếng Anh và tạo nhãn `.txt` chuẩn YOLO.
  3. *Auto Training & Export:* Sử dụng SDK `ultralytics` huấn luyện `yolov8n.pt` và tự động export `.onnx`.
  4. *Tích hợp CLI:* Mở rộng `Scaffold` hỗ trợ lệnh:
     `python main.py train --task=butterfly --type=area_select`

### B. Dạng 2: `image_drag_drop` (Kéo thả mảnh ghép / Khớp vị trí)

- **Mô tả bài toán:** Kéo thả mảnh ghép (puzzle piece) vào khoảng trống thiếu trên bức ảnh hoặc kéo vật thể đến mục tiêu chỉ định.
- **Mô hình mục tiêu:**
  - **Feature Matching / Template Matching (SuperPoint + LightGlue hoặc OpenCV FlannBasedMatcher)** đối với Puzzle Drag-Drop.
  - **Keypoint Detection / Spatial Regression (YOLOv8-Pose hoặc Custom CNN Regression)** đối với kéo vật thể đến đích.
- **Giải pháp kiến trúc đề xuất:**
  1. *Dataset Generator:* Tạo `src/components/dataset_gen/drag_drop.py` tự động cắt ngẫu nhiên các mảnh ghép từ ảnh nền và tạo nhãn tọa độ gốc $(x, y)$.
  2. *Matching & Training Engine:* Sử dụng mô hình trích xuất đặc trưng điểm (Keypoint matching) để tính độ tương đồng không gian và tìm vị trí tọa độ $(x_0, y_0)$ có điểm tương đồng cao nhất.
  3. *Export ONNX:* Xuất mô hình trích xuất feature hoặc regression model sang file `.onnx`.

---

## 3. Đánh giá tính khả thi & Kết luận

| Tiêu chí | `image_label_area_select` | `image_drag_drop` |
| :--- | :--- | :--- |
| **Mức độ khả thi** | 🟢 **Rất Cao (95%)** | 🟡 **Khá Cao (85%)** |
| **Nền tảng sẵn có trong code** | Đã có sẵn stub `TaskType` & `dataset_gen/` & script eval YOLO | Cần viết thêm bộ cắt phôi & matching |
| **Công nghệ cốt lõi** | Ultralytics YOLOv8/v11 + YOLO-World | SuperPoint / LightGlue / Template Matching / CNN Regression |
| **Kích thước file ONNX xuất ra** | ~6MB - 12MB (YOLOv8n / YOLOv8s) | ~2MB - 10MB |

**KẾT LUẬN:** 
**HOÀN TOÀN CÓ THỂ NÂNG CẤP ĐƯỢC.** Dự án đã có sẵn nền móng kiến trúc `ModelFactory` mở và các mã nguồn thử nghiệm YOLO. Nâng cấp này sẽ giúp `hcaptcha-model-factory` bao phủ 100% các dạng bài toán hCaptcha phổ biến hiện nay.
