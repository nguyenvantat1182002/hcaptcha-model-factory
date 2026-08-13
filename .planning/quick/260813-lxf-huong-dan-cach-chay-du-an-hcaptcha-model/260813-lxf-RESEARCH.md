# Research Findings: Cách chạy dự án hcaptcha-model-factory

**Task ID:** 260813-lxf  
**Date:** 2026-08-13  
**Target:** Hướng dẫn chi tiết cách cài đặt, cấu hình và vận hành dự án `hcaptcha-model-factory`.

---

## 1. Cấu trúc và Điểm khởi chạy chính (Entry Points)

- **CLI Scaffold chính:** [src/main.py](file:///d:/Python/hcaptcha-model-factory/src/main.py)
  - Sử dụng thư viện `google-fire` để liên kết với lớp `Scaffold` ([src/apis/scaffold.py](file:///d:/Python/hcaptcha-model-factory/src/apis/scaffold.py)).
  - Hỗ trợ các lệnh: `new`, `train`, `val`, `test_onnx`, `trainval`.
- **Automation Pipeline:** [automation/_04_mini_workflow.py](file:///d:/Python/hcaptcha-model-factory/automation/_04_mini_workflow.py)
  - Quy trình tự động hóa tải dữ liệu, train, tạo version và upload release lên GitHub repository (`QIN2DIM/hcaptcha-challenger`).
- **Web Dashboard (Frontend):** [frontend/](file:///d:/Python/hcaptcha-model-factory/frontend/)
  - Giao diện quản trị built trên Reflex framework (`rxconfig.py`).
- **Evaluation Scripts:** [evaluation/](file:///d:/Python/hcaptcha-model-factory/evaluation/)
  - Kiểm thử mô hình ONNX đã xuất với OpenCV DNN (`eva_resnet_cls_model.py`).

---

## 2. Các lệnh chạy cơ bản

### Bước 1: Chuẩn bị môi trường Python
```bash
# Kích hoạt môi trường ảo (Virtualenv đã có sẵn ở .venv)
.\.venv\Scripts\activate       # Trên Windows (PowerShell)
# hoặc source .venv/bin/activate trên Linux/macOS

# Cài đặt phụ thuộc
pip install -r requirements.txt
# Hoặc nếu dùng uv:
uv sync
```

### Bước 2: Chạy Quy trình Scaffold đầy đủ (Tạo task + Auto-Labeling + Training)
```bash
cd src
python main.py new
```
**Quy trình tương tác:**
1. Nhập prompt tiếng Anh của thử thách hCaptcha (ví dụ: `Please click each image containing a smiling dog`).
2. Hệ thống tự tạo thư mục `data/smiling_dog/unlabel` và tự mở thư mục này trong OS File Explorer.
3. Người dùng chép các file ảnh chưa dán nhãn vào thư mục `unlabel/`.
4. Nhấn phím bất kỳ tại console. AI sẽ trích xuất vector đặc trưng (ResNet/CLIP) và phân cụm (PCA + K-Means) đưa ảnh vào `yes/` (ảnh chứa đối tượng) và `bad/` (ảnh không chứa đối tượng).
5. Người dùng kiểm tra thủ công trong `yes/` và `bad/` để điều chỉnh nếu có sai sót.
6. Xác nhận `y` để bắt đầu huấn luyện PyTorch `ResNetMini` và xuất mô hình file `.onnx` ra thư mục `model/smiling_dog/`.

### Bước 3: Huấn luyện / Kiểm thử mô hình đã có sẵn dữ liệu
```bash
cd src

# Train và Validation kết hợp
python main.py trainval --task=smiling_dog

# Chỉ Train (tùy chỉnh epochs & batch_size)
python main.py train --task=smiling_dog --epochs=100 --batch_size=8

# Chỉ Validation
python main.py val --task=smiling_dog

# Kiểm thử inference file ONNX xuất ra
python main.py test_onnx --task=smiling_dog --flag=all
```

### Bước 4: Chạy Dashboard Frontend (Reflex UI)
```bash
cd frontend
pip install reflex
reflex run
```

---

## 3. Các lưu ý & Bẫy thường gặp (Pitfalls & Gotchas)

1. **Lệnh `python main.py` ở thư mục gốc (root):**
   - File `main.py` ở thư mục gốc chỉ chứa hàm in `"Hello from hcaptcha-model-factory!"`. Để dùng CLI, bạn **phải `cd src`** hoặc chạy `python src/main.py <command>`.
2. **Ký tự đặc biệt trong Tên Task (--task):**
   - Các ký tự dấu cách, dấu phẩy, dấu gạch ngang sẽ được tự động đổi thành dấu gạch dưới `_`.
   - Ví dụ: `--task="smiling dog"` -> `smiling_dog`.
3. **Phụ thuộc OpenCV & CUDA:**
   - Nếu muốn huấn luyện GPU, cần cài PyTorch bản CUDA: `pip install torch torchvision --index-url https://download.pytorch.org/whl/cu118`.
