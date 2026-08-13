# Coding Conventions

**Analysis Date:** 2026-08-13

## Naming Patterns

**Files:**
- `snake_case.py` for standard Python modules (e.g., `scaffold.py`, `resnet_mini.py`, `focal_loss.py`).
- Numbered prefix `_0N_name.py` for ordered automation step scripts (e.g., `_01_datasets_downloader.py`, `_04_mini_workflow.py`).
- Private / internal helper scripts prefixed with underscore `_name.py` (e.g., `_annotator.py`, `_flow_card.py`).

**Classes:**
- `PascalCase` for classes and dataclasses (e.g., `Scaffold`, `ModelFactory`, `ResNet`, `ResNetMini`, `ResidualBlock`, `DataModel`, `ClusterLabeler`).
- Enum classes in `PascalCase` with UPPER_SNAKE_CASE members (e.g., `NestedPrompts`, `ModelVersion` in `automation/_04_mini_workflow.py`).

**Functions & Methods:**
- `snake_case` for function and method names (e.g., `diagnose_task()`, `conv_pth2onnx()`, `_build_env()`, `binary_classify()`).
- Private / helper methods prefixed with single underscore `_method_name()` (e.g., `_archive_previous_models()`, `_make_datamodel()`).

**Variables & Constants:**
- `snake_case` for local variables and parameters.
- `UPPER_SNAKE_CASE` for class attributes and module-level constants (e.g., `BATCH_SIZE`, `DIR_MODEL`, `FILENAME_YAML_ALL`, `GH_MODELHUB_TITLE`).

## Code Style

**Formatting:**
- Standard Python 4-space indentation.
- PEP 8 compliant code structure with strict type hint annotations where applicable (`typing.Optional`, `typing.Dict`, `Path`, `BaseModel`).

**Imports:**
- Standard library imports first (e.g., `os`, `sys`, `shutil`, `time`, `typing`).
- Third-party packages second (e.g., `torch`, `torchvision`, `cv2`, `numpy`, `loguru`, `pydantic`, `fire`, `yaml`).
- Internal module imports third (e.g., `from components.dataset import BinaryDataset`, `from factories.resnet import ResNet`).

**Dataclasses & Models:**
- Standard `@dataclass` used for dataset state in `src/factories/kernel.py` (`DataModel`).
- `pydantic.BaseModel` with `@field_validator` used for workflow parameter validation in `automation/_04_mini_workflow.py` (`WorkFlow`).

## Error Handling & Task Input Normalization

**Input Normalization (`diagnose_task` in `src/apis/scaffold.py`):**
- Converts space/comma/hyphen separators to underscores `_`.
- Strips leading and trailing whitespace.
- Checks for invalid path characters (`\`, `/`, `:`, `*`, `?`, `<`, `>`, `|`) and raises `TypeError`.
- Normalizes bad homoglyph / Cyrillic characters (e.g. converting Cyrillic 'а', 'е', 'о' to ASCII 'a', 'e', 'o').

**Logging with Loguru:**
- Use `loguru.logger` for all log output (`logger.debug()`, `logger.success()`, `logger.catch`).
- Use `@logger.catch` decorator on top-level API methods (`Scaffold.train`, `Scaffold.val`, `Scaffold.new`) to capture and log unhandled exceptions gracefully.

**Defensive Checks:**
- Explicit file existence and resource size checks before training starts (e.g. raising `FileNotFoundError` or `ResourceWarning` if positive/negative dataset folders are incomplete or missing).

---

*Conventions analysis: 2026-08-13*
*Update after changing coding standards or adding linter configs*
