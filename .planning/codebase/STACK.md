# Technology Stack

**Analysis Date:** 2026-08-13

## Languages

**Primary:**
- Python >=3.12 - Core application logic, PyTorch neural network definition, model training, dataset generation, auto-labeling, and CLI interface (`pyproject.toml`, `.python-version`)

**Secondary:**
- YAML - Dataset split manifests (`all.yaml`, `train.yaml`, `val.yaml`, `test.yaml`) and workflow configuration (`server/onnx-flow.yaml`)
- Markdown - Documentation, wiki pages, and planning specifications (`README.md`, `hcaptcha-model-factory.wiki/`)

## Runtime

**Environment:**
- Python 3.12+ virtual environment (`.venv/`)
- CUDA / GPU environment optional for accelerated PyTorch training (`torch.cuda` detected dynamically in `src/factories/kernel.py`)

**Package Manager:**
- `uv` (Lockfile: `uv.lock` present)
- `pip` (`pyproject.toml` and `requirements.txt` present)

## Frameworks

**Core:**
- PyTorch (`torch>=2.0.1`, `torchvision>=0.15.2`) - Deep learning framework for neural network training and data augmentation (`src/factories/resnet.py`, `src/components/nn/resnet_mini.py`)
- ONNX Runtime (`onnx>=1.19.0`, `opencv-python~=4.8.0.76`) - ONNX model export and OpenCV DNN execution (`src/factories/kernel.py`, `evaluation/eva_resnet_cls_model.py`)
- Google Fire (`fire>=0.7.1`) - CLI interface generator (`src/main.py`)
- Pydantic (`pydantic~=2.5.0`) - Data validation and setting management (`automation/_04_mini_workflow.py`)
- Reflex (`frontend/`) - Python full-stack web framework (`frontend/rxconfig.py`, `frontend/README.md`)

**Testing & Evaluation:**
- In-repo evaluation scripts using `cv2.dnn` and `hcaptcha_challenger` (`evaluation/eva_resnet_cls_model.py`, `evaluation/eva_challenger.py`)
- No standard unit testing framework integrated in dependencies

**Build/Dev:**
- `uv` / standard setuptools / hatchling for dependency resolution
- Jupyter Notebooks (`automation/roboflow_resnet.ipynb`)

## Key Dependencies

**Critical:**
- `torch` & `torchvision` (`torch>=2.0.1`, `torchvision>=0.15.2`) - Deep learning framework and vision transforms (`src/factories/resnet.py`)
- `onnx` (`onnx>=1.19.0`) - Model serialization format for production deployment (`src/factories/kernel.py`)
- `opencv-python` (`opencv-python~=4.8.0.76`) - Image processing and ONNX runtime inference via `cv2.dnn` (`evaluation/eva_resnet_cls_model.py`)
- `scikit-learn` (`scikit-learn>=1.9.0`) - Clustering algorithms for unsupervised auto-labeling (`src/components/auto_label/cluster.py`)
- `pillow` (`pillow>=12.3.0`) - Image loading and manipulation (`evaluation/eva_resnet_cls_model.py`)

**Infrastructure & Utilities:**
- `loguru` (`loguru>=0.7.3`) - Logging framework across application components (`src/apis/scaffold.py`, `src/factories/resnet.py`)
- `pyyaml` (`pyyaml>=6.0.3`) - YAML dataset metadata parsing and dumping (`src/factories/kernel.py`)
- `PyGithub` - GitHub REST API interaction for releases and asset deployment (`automation/_04_mini_workflow.py`)

## Configuration

**Environment:**
- Environment variable `GITHUB_TOKEN` for GitHub release deployment (`automation/_04_mini_workflow.py`)
- CLI parameters passed via Google Fire (`src/main.py`)

**Build:**
- `pyproject.toml` - Python project metadata and dependencies
- `requirements.txt` - Alternative dependency declaration
- `.python-version` - Specified Python version (3.12)

## Platform Requirements

**Development:**
- Windows / macOS / Linux with Python 3.12+
- Optional CUDA 11.8+ GPU environment for faster training (`requirements.txt`)

**Production:**
- Exported ONNX models (`.onnx`) consumed by `hcaptcha-challenger` downstream library

---

*Stack analysis: 2026-08-13*
*Update after major dependency changes*
