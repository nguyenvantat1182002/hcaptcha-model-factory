# Architecture

**Analysis Date:** 2026-08-13

## Pattern Overview

**Overall:** Modular Deep Learning Factory & Scaffolding Pipeline (CLI & Automation Engine)

**Key Characteristics:**
- **Scaffold-driven CLI:** Entry point exposes high-level workflow commands (`new`, `train`, `val`, `test_onnx`, `trainval`) via Google Fire.
- **Abstract Model Factory Pattern:** Base `ModelFactory` class enforces lifecycle stages (environment build, dataset split, model archiving, ONNX conversion).
- **Unsupervised Auto-Labeling Pipeline:** Feature extraction (CNN / CLIP embeddings) + clustering (K-Means/scikit-learn) for rapid dataset preparation without manual annotation.
- **Decoupled Neural Network Architecture:** Lightweight CNN (`ResNetMini`) parameterized independently from training logic.

## Layers

**1. API & CLI Interface Layer (`src/apis/scaffold.py`, `src/main.py`)**
- **Purpose:** Exposes public user commands and orchestrates interactive setup flows.
- **Contains:** `Scaffold` class, `diagnose_task()` string normalization helper, `fire.Fire` binder.
- **Depends on:** `components.auto_label`, `components.config`, `components.utils`, `factories.resnet`.
- **Used by:** Command-line users running `python src/main.py <command>`.

**2. Factory Abstraction Layer (`src/factories/kernel.py`, `src/factories/resnet.py`)**
- **Purpose:** Encapsulates end-to-end model lifecycle (data management, training loops, validation, export).
- **Contains:** `ModelFactory` abstract base class, `DataModel` dataclass, `ResNet` concrete factory implementation.
- **Depends on:** `components.dataset`, `components.losses`, `components.nn`, `components.utils`.
- **Used by:** `Scaffold` API layer and automation scripts (`automation/_04_mini_workflow.py`).

**3. Core Components Layer (`src/components/`)**
- **Purpose:** Reusable building blocks for neural networks, losses, datasets, auto-labeling, and utilities.
- **Sub-components:**
  - `components/nn/`: Network architectures (`ResNetMini`, `ResidualBlock`).
  - `components/auto_label/`: Feature extraction and clustering (`ClusterLabeler`, `img2emb`, `cluster.py`).
  - `components/dataset/`: PyTorch dataset definitions (`BinaryDataset`, `universal.py`).
  - `components/dataset_gen/`: Dataset generation logic (`on_select.py`, `on_select_animal.py`, `on_select_digit.py`).
  - `components/losses/`: Custom loss functions (`FocalLoss`).
  - `components/utils/`: Shared utilities (`ToolBox`, `Config`).
- **Used by:** Factory layer and standalone scripts.

**4. Automation & Deployment Layer (`automation/`, `evaluation/`)**
- **Purpose:** Automated batch training pipelines, release deployment via GitHub API, asset manager, and benchmark evaluation against ONNX/challenger targets.
- **Contains:** `WorkFlow` Pydantic model (`_04_mini_workflow.py`), dataset downloader, asset manager, evaluation scripts (`eva_resnet_cls_model.py`).

**5. Presentation Layer (`frontend/`)**
- **Purpose:** Web dashboard UI built with Reflex Python framework.
- **Contains:** Reflex app configuration (`rxconfig.py`), components, pages, and state definitions.

## Data Flow

**Standard Training & Export Lifecycle:**

```
[ User Input / Task Name ]
          │
          ▼
[ Scaffold.train(task="bird_flying") ]
          │
          ▼
[ ResNet(ModelFactory) Initialization ]
   ├── Archive old models to model/[task]/[timestamp]/
   └── Build dataset environment (_build_env)
          │  ├── Scan dataset/yes/ and dataset/bad/
          │  └── Random split into train.yaml (80%) and val.yaml (20%)
          ▼
[ ResNet.train() Execution ]
   ├── Load PyTorch BinaryDataset with image augmentations
   ├── Train ResNetMini using FocalLoss / Adam optimizer
   └── Save PyTorch weights (.pth)
          │
          ▼
[ ResNet.conv_pth2onnx() ]
   └── Export trained PyTorch model to ONNX binary model/[task]/[task].onnx
          │
          ▼
[ Downstream Consumption / Release ]
   └── Used by hcaptcha-challenger or deployed via GitHub Release
```

**Unsupervised Auto-Labeling Lifecycle:**

```
[ Raw Unlabeled Images in unlabel/ ]
          │
          ▼
[ Scaffold.new() -> ClusterLabeler.run() ]
          │
          ▼
[ img2emb Feature Extraction ] ──► [ Feature Vector Embeddings ]
                                             │
                                             ▼
                                  [ Scikit-Learn Clustering ]
                                             │
                                             ▼
                                  [ Split into yes/ & bad/ ]
```

## Key Abstractions

- `ModelFactory` (`src/factories/kernel.py`): Foundation class providing dataset tracking (`DataModel`), directory management, environment setup, and model archiving.
- `ResNetMini` (`src/components/nn/resnet_mini.py`): Custom 2-block ResNet CNN taking 64x64 RGB images and returning class logits.
- `FocalLoss` (`src/components/losses/focal_loss.py`): Loss function implementation addressing class imbalance in binary classification tasks.
- `BinaryDataset` (`src/components/dataset/binary.py`): Custom PyTorch `Dataset` wrapper loading images specified in split YAML files (`train.yaml`, `val.yaml`).
- `ToolBox` (`src/components/utils/toolbox/toolbox.py`): Utility collection for prompt splitting, image format validation, and path operations.

---

*Architecture analysis: 2026-08-13*
*Update after major structural or architectural changes*
