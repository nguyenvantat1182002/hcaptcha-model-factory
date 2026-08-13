# Research Findings: Tích hợp FunCaptcha (Arkose Labs) vào Model Factory

**Task ID:** 260813-m53  
**Date:** 2026-08-13  
**Target:** Đánh giá khả năng mở rộng kiến trúc hcaptcha-model-factory để tự động tạo và huấn luyện mô hình giải bài toán FunCaptcha (Arkose Labs Captcha).

---

## 1. Phân tích bài toán FunCaptcha vs. hCaptcha

| Đặc điểm | hCaptcha | FunCaptcha (Arkose Labs) |
| :--- | :--- | :--- |
| **Định dạng ảnh** | Lưới 3x3 / 4x4 (64x64px) hoặc ảnh đơn | Khung tròn / 6 vị trí xoay / 6 ảnh lựa chọn |
| **Dạng bài toán** | Phân loại nhị phân (Yes/Bad), Bounding Box | Khớp xoay góc (Rotation), Phân loại 1 trong N (Pick 1 of 6), Đếm điểm xúc xắc (Dice math) |
| **Đầu ra mong muốn** | Mảng boolean `[True, False, ...]` hoặc Bounding Box | Chỉ số xoay góc ($0^\circ-360^\circ$) hoặc Index đáp án đúng ($0..5$) |

---

## 2. Các dạng thử thách FunCaptcha phổ biến & Mô hình tương ứng

1. **Dạng 1: Rotation / Alignment (Xoay con vật/vật thể theo chỉ tay):**
   - *Đặc điểm:* Xoay vật thể trong khung hình theo góc $0^\circ, 60^\circ, 120^\circ, 180^\circ, 240^\circ, 300^\circ$ (6 góc) hoặc xoay tự do $0^\circ-360^\circ$.
   - *Mô hình phù hợp:* **ResNetMini / EfficientNet (Multi-class Classification 6 classes)** hoặc **Angle Regression Model (Predict Continuous Angle $\theta$)**.
2. **Dạng 2: Pick 1 of 6 (Chọn 1 trong 6 hình thỏa mãn điều kiện):**
   - *Đặc điểm:* Ví dụ: "Chọn hình hai con vật giống nhau", "Chọn hình tổng xúc xắc bằng 8", "Chọn con vật đứng đúng chiều".
   - *Mô hình phù hợp:* **Pair Matching Network (Siamese Net)** hoặc **ViT / ResNet Classifier** dự đoán xác suất từng ô trong 6 ô.
3. **Dạng 3: 3D Animal Standing Upright (Chọn hình con vật đứng thẳng):**
   - *Đặc điểm:* 6 hình chụp mô hình 3D xoay ở các góc khác nhau, chọn hình duy nhất con vật đứng đúng chiều thẳng đứng.
   - *Mô hình phù hợp:* **Classification Network (6-class softmax)**.

---

## 3. Tái sử dụng kiến trúc `hcaptcha-model-factory` (Reusability)

Kiến trúc hiện tại của dự án có khả năng **tái sử dụng 80-85%**:

1. **Khung `ModelFactory` ([src/factories/kernel.py](file:///d:/Python/hcaptcha-model-factory/src/factories/kernel.py)):**
   - Quản lý vòng đời mô hình, tạo tập dữ liệu YAML (`train.yaml`, `val.yaml`), tự động lưu vết và xuất ONNX hoàn toàn tái sử dụng được.
2. **Lớp mô hình PyTorch `ResNetMini` ([src/components/nn/resnet_mini.py](file:///d:/Python/hcaptcha-model-factory/src/components/nn/resnet_mini.py)):**
   - Chỉ cần điều chỉnh `num_classes` từ 2 (binary) thành 6 (cho 6 góc xoay hoặc 6 ô lựa chọn FunCaptcha).
3. **Quy trình Auto-Labeling ([src/components/auto_label/](file:///d:/Python/hcaptcha-model-factory/src/components/auto_label/)):**
   - Trích xuất Feature Embeddings và K-Means clustering giúp tự động gom các góc xoay giống nhau mà không cần gán nhãn thủ công.

---

## 4. Các điểm cần thay đổi/bổ sung (Changes Needed)

1. **Cắt ảnh tiền xử lý (FunCaptcha Image Splitter):**
   - Thêm `src/components/utils/funcaptcha_parser.py`: Tự động cắt các bức ảnh FunCaptcha 6 khung tròn thành 6 ảnh đơn $128\times 128$ độc lập.
2. **Chuyển đổi dự án thành `captcha-model-factory` (Generic Framework):**
   - Đổi tên hoặc tái cấu trúc Scaffold CLI để hỗ trợ cả hCaptcha và FunCaptcha:
     ```bash
     python main.py train --provider=funcaptcha --task=rotate_animal --classes=6
     python main.py train --provider=hcaptcha --task=smiling_dog
     ```
3. **Loss Function cho bài toán Xoay (Angle Regression / CrossEntropy):**
   - Bài toán xoay góc có thể dùng `CrossEntropyLoss` (với 6 góc $60^\circ$) hoặc `MSELoss` (cho góc quay liên tục).

---

## 5. Kết luận

Tích hợp FunCaptcha là một bước nâng cấp **RẤT TỰ NHIÊN VÀ ĐÁNG GIÁ**. Kiến trúc factory cốt lõi của dự án hoàn toàn đủ linh hoạt để biến repo này thành **Một hệ thống Model Factory đa năng (Universal Captcha Model Factory)** giải quyết được cả hCaptcha, reCaptcha và FunCaptcha (Arkose Labs).
