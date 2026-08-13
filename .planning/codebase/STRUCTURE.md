# Codebase Structure

**Analysis Date:** 2026-08-13

## Directory Layout

```
hcaptcha-model-factory/
├── .agents/                    # GSD workflows, agent skills, and system rules
├── .github/                    # Issue templates, PR templates, and GitHub Actions CI/CD workflows
├── automation/                 # Dataset downloaders, asset management, and automated release workflows
├── evaluation/                 # Model evaluation scripts for ResNet, YOLO, CLIP, and challenger
├── frontend/                   # Reflex-based web frontend application
├── hcaptcha-model-factory.wiki/# GitHub Wiki submodule containing documentation pages
├── server/                     # Server flow configuration specs (onnx-flow.yaml)
├── src/                        # Main Python source code directory
│   ├── apis/                   # Public Scaffold API and CLI command implementation
│   ├── components/             # Reusable core modules (NN models, datasets, loss functions, auto-labeling)
│   └── factories/              # Model factory kernel base class and ResNet factory
├── main.py                     # Root entrypoint stub
├── pyproject.toml              # Project dependency and build metadata
├── requirements.txt            # Alternative pip requirements specification
├── uv.lock                     # Lockfile for uv package manager
└── README.md                   # Repository README overview
```

## Directory Purposes

**`src/`:**
- Purpose: Primary application codebase containing core neural network definitions, training factory pipeline, CLI commands, auto-labeling, and utilities.
- Contains: `.py` Python modules.
- Subdirectories:
  - `src/apis/`: Scaffolding API (`scaffold.py`).
  - `src/factories/`: Model factory base (`kernel.py`) and ResNet implementation (`resnet.py`).
  - `src/components/`: Core components sub-modules:
    - `src/components/auto_label/`: Unsupervised clustering (`cluster.py`, `emb.py`, `img2emb.py`, `base.py`).
    - `src/components/dataset/`: PyTorch dataset definitions (`binary.py`, `universal.py`).
    - `src/components/dataset_gen/`: Dataset generation logic (`on_select.py`, `on_select_animal.py`, `on_select_digit.py`).
    - `src/components/losses/`: Loss functions (`focal_loss.py`).
    - `src/components/nn/`: Neural network architectures (`resnet_mini.py`).
    - `src/components/utils/`: Shared utilities (`config.py`, `toolbox/toolbox.py`).

**`automation/`:**
- Purpose: Automated dataset downloading, asset management, auto-labeling workflow execution, and GitHub Release model deployment.
- Key files: `_04_mini_workflow.py`, `_03_auto_labeling.py`, `_02_assets_manager.py`, `_01_datasets_downloader.py`, `_flow_card.py`, `roboflow_resnet.ipynb`.

**`evaluation/`:**
- Purpose: Benchmarking and evaluation scripts for trained ONNX models against test datasets and competitor models.
- Key files: `eva_resnet_cls_model.py`, `eva_challenger.py`, `eva_clip_model.py`, `eva_yolo_det_model.py`, `eva_yolo_seg_model.py`.

**`frontend/`:**
- Purpose: Web user interface dashboard built with Reflex framework.
- Key files: `rxconfig.py`, `README.md`, `frontend/frontend.py`, `frontend/state.py`, `frontend/styles.py`.

**`server/`:**
- Purpose: Server runtime flow configuration files.
- Key files: `onnx-flow.yaml`, `readme.md`.

**`.github/`:**
- Purpose: GitHub repository configuration, templates, and CI GitHub Actions.
- Key files: `workflows/codeql-analysis.yml`, `workflows/release-draft.yml`, `pull_request_template.md`.

## Key File Locations

**Entry Points:**
- `src/main.py`: Primary CLI application entry point using Google Fire.
- `main.py`: Root stub file.
- `automation/_04_mini_workflow.py`: Standalone CLI automation script for full training and release pipeline.
- `frontend/frontend/frontend.py`: Reflex web UI entry point.

**Configuration:**
- `pyproject.toml`: Python project metadata and dependencies.
- `requirements.txt`: Requirements list for pip installation.
- `.python-version`: Python version constraint (3.12).
- `src/components/config.py`: Directory path constants and configuration settings.
- `server/onnx-flow.yaml`: Server workflow specifications.

**Core Logic:**
- `src/apis/scaffold.py`: Public Scaffold API methods (`new`, `train`, `val`, `test_onnx`, `trainval`).
- `src/factories/kernel.py`: Base `ModelFactory` and `DataModel` structures.
- `src/factories/resnet.py`: ResNet training, dataset splitting, validation, and ONNX conversion logic.
- `src/components/nn/resnet_mini.py`: PyTorch `ResNetMini` architecture and `ResidualBlock`.
- `src/components/losses/focal_loss.py`: PyTorch `FocalLoss` implementation.
- `src/components/dataset/binary.py`: `BinaryDataset` PyTorch class.

---

*Structure analysis: 2026-08-13*
*Update after directory reorganization or new module additions*
