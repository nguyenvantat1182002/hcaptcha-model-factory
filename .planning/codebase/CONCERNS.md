# Codebase Concerns

**Analysis Date:** 2026-08-13

## Tech Debt

**Dual Root & Subdirectory Entrypoints:**
- Issue: `main.py` at the project root is a stub containing only `print("Hello from hcaptcha-model-factory!")`, whereas the actual CLI application entry point is `src/main.py`.
- Why: Root file created during initialization or uv setup without binding `fire.Fire(Scaffold)`.
- Impact: Running `python main.py` at root does nothing useful and confuses users.
- Fix approach: Delegate root `main.py` to import and execute `src/main.py` or invoke `Fire(Scaffold)`.

**Hardcoded Task Paths in Automation & Evaluation:**
- Issue: Evaluation scripts contain hardcoded target model and image paths (e.g. `model_zoo/attention_bus.onnx`, `zip_dir/challenge_bus`, `database2309`).
- Why: Quick script creation during dataset iterations.
- Impact: Running evaluation scripts without pre-created dummy folders causes `FileNotFoundError`.
- Fix approach: Parameterize paths using CLI arguments or environment variables via Pydantic/Fire.

**Fixed Image Resolution & Hyperparameters in Base Dataclass:**
- Issue: `DataModel` in `src/factories/kernel.py` hardcodes `img_size: 64` in `__post_init__`, and `ResNetMini` expects fixed 64x64 input dimensions.
- Why: Default hCaptcha image cell size assumption.
- Impact: Restricts application to 64x64 inputs unless code is modified across multiple components (`resnet.py`, `binary.py`, `resnet_mini.py`).
- Fix approach: Expose `img_size` as a configurable parameter in `ModelFactory` and neural network constructors.

## Known Bugs & Fragile Areas

**Interactive Stdin Prompts in `Scaffold.new()`:**
- Issue: `Scaffold.new()` in `src/apis/scaffold.py` uses blocking `input()` calls and opens native OS file manager windows (`os.startfile()` or `xdg-open`).
- Trigger: Running `python src/main.py new` in non-interactive CI/CD environments.
- Symptoms: Process hangs indefinitely waiting for user keyboard input.
- Workaround: Avoid using `new()` command in CI; use non-interactive automation scripts like `automation/_04_mini_workflow.py`.

**Inconsistent Dependency Definitions:**
- Issue: Differences between dependencies declared in `pyproject.toml` (`numpy~=1.26.0`, `pillow>=12.3.0`, `torch>=2.0.1`) and `requirements.txt`.
- Risk: Inconsistent package resolution depending on whether `uv` or `pip install -r requirements.txt` is used.
- Recommendations: Synchronize `requirements.txt` with `pyproject.toml`.

## Security Considerations

**GitHub Personal Access Tokens:**
- Risk: `GITHUB_TOKEN` environment variable used in `automation/_04_mini_workflow.py` for repository deployment.
- Mitigation: Read from environment variables rather than hardcoded strings. Ensure tokens are never printed in logs or committed to git.

## Performance Bottlenecks

**Sequential Image Transformation & CPU Loading:**
- Problem: Large datasets with extensive data augmentation (`torchvision.transforms.RandomChoice`) on CPU can slow down GPU training batches.
- Impact: Lower GPU utilization during epoch iterations.
- Recommendations: Increase DataLoader worker threads (`num_workers`) in `ResNet.train()` when CUDA is available.

---

*Concerns analysis: 2026-08-13*
*Update after addressing tech debt or fixing known issues*
