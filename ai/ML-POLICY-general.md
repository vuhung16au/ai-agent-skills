# ML-POLICY-general.md

## Purpose

Define standards for building reproducible, maintainable, and trustworthy machine learning code. This policy governs agent behavior when generating ML pipelines, training scripts, evaluation code, and experiment infrastructure.

---

## Scope / When to Use

Apply this policy to all ML code generation, including:
- Dataset loading, preprocessing, and splitting.
- Model definition, training, and evaluation.
- Experiment tracking and logging.
- Inference pipelines and model serving scaffolding.

For deep learning framework-specific rules, supplement with a framework-specific policy.

---

## Core Principles

- **Reproducibility over convenience.** Every experiment must be reproducible from code and config alone.
- **No data leakage.** Validation and test sets must remain unseen until evaluation.
- **Honest evaluation.** Report the right metrics for the right split on the right task.
- **Modular pipelines.** Separate data, model, training, and evaluation concerns.
- **Track everything.** If it is not logged, it did not happen.

---

## Required Behavior

### Reproducibility
- Set random seeds for all sources of randomness: Python `random`, `numpy`, framework-specific RNGs (e.g., `torch.manual_seed`, `tf.random.set_seed`).
- Pin all dependency versions in `requirements.txt` or `pyproject.toml`.
- Store configuration (hyperparameters, data paths, model architecture choices) in a config file (YAML, TOML, or JSON) — not hardcoded in scripts.
- Record the Git commit hash alongside each experiment run.

```python
import random
import numpy as np

SEED = 42
random.seed(SEED)
np.random.seed(SEED)
# torch.manual_seed(SEED)  # uncomment for PyTorch
```

### Dataset Handling
- Always perform train/validation/test splits before any preprocessing that could cause leakage (e.g., normalization, imputation).
- Fit preprocessing transformers (scalers, encoders, imputers) on training data only; transform validation and test sets using the fitted transformer.
- Never use the test set during model selection or hyperparameter tuning.
- Document the dataset source, version, and any known limitations in a comment or docstring.

### Model Definition
- Define models in dedicated modules or classes, separate from training logic.
- Accept hyperparameters as explicit arguments — do not hardcode them in the model definition.
- Include a brief docstring describing the model architecture and expected input shape.

### Training
- Log training loss and validation metrics at every epoch or meaningful interval.
- Implement early stopping or a maximum epoch limit to prevent runaway training.
- Save checkpoints at regular intervals and at the best validation metric.
- Avoid training on the full dataset before hyperparameter tuning is complete.

### Evaluation
- Evaluate final models on the held-out test set only once.
- Report multiple relevant metrics (accuracy alone is insufficient for imbalanced datasets).
- Include confidence intervals or standard deviations when reporting results from multiple runs.
- Be explicit about what "best model" means (e.g., best validation loss, best F1 score).
- Never report test set metrics during development iteration.

### Experiment Tracking
- Use an experiment tracking tool (MLflow, Weights & Biases, DVC, or similar).
- Log: hyperparameters, dataset version/hash, model architecture summary, all evaluation metrics, and artifact paths.
- Assign a meaningful name or tag to each experiment run.
- Do not rely on print statements as the primary logging mechanism.

### Pipeline Structure
- Separate concerns into distinct pipeline stages: data → features → train → evaluate → serve.
- Each stage should be callable independently and accept explicit inputs/outputs (file paths or in-memory objects, not global state).
- Use a pipeline orchestration tool (e.g., `sklearn.Pipeline`, DVC, Prefect, Kedro) for multi-stage workflows.

### Inference
- Serialize the complete inference pipeline (preprocessor + model) together, not just model weights.
- Document input schema (feature names, types, expected ranges) and output schema.
- Validate inputs at inference time before passing them to the model.

---

## Things to Avoid

- Fitting any preprocessor on validation or test data.
- Reporting test set metrics more than once per model candidate.
- Hardcoding hyperparameters, file paths, or random seeds in training scripts.
- Using `accuracy` as the sole metric for imbalanced classification problems.
- Ignoring class imbalance, missing data, or distribution shift in preprocessing.
- Committing large datasets or model weights to Git — use DVC, LFS, or cloud storage.
- Running experiments without tracking the configuration and results.
- Mixing data loading, model logic, and evaluation in a single monolithic script.

---

## Output Expectations

Generated ML code must:
- Set random seeds explicitly.
- Split data before any preprocessing.
- Fit preprocessors on train data only.
- Log hyperparameters and metrics to an experiment tracker.
- Separate model definition from training logic.
- Include type annotations and docstrings on public functions.
- Not hardcode file paths, credentials, or hyperparameters.

---

## Optional Checklist

- [ ] Random seeds set for all RNG sources.
- [ ] Dependencies pinned in requirements file.
- [ ] Hyperparameters in a config file, not hardcoded.
- [ ] Train/val/test split performed before preprocessing.
- [ ] Preprocessors fitted on train data only.
- [ ] Test set used for final evaluation only.
- [ ] Experiment tracker configured.
- [ ] Metrics include more than just accuracy.
- [ ] Model checkpoint saving implemented.
- [ ] Inference pipeline serializes preprocessor + model together.
