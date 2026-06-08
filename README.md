# RSNA Pneumonia Final Experiment

## Overview

This repository contains a Google Colab workflow for the RSNA Pneumonia Detection Challenge. The notebook is written as a two-part experimental pipeline:

1. a hold-out experiment with train/validation/test splitting, and
2. a 5-fold cross-validation experiment.

The notebook is based on PNG images converted from DICOM files and focuses on comparing a standalone student model with two knowledge-distillation variants.

## Notebook Scope

The shared notebook includes:

- Kaggle dataset download commands
- DICOM-to-PNG preprocessing
- PNG-based dataset loading
- train/validation/test splitting
- 5-fold stratified cross-validation
- baseline model training
- knowledge distillation training
- metrics, plots, and CSV export
- optional Google Drive backup for CV outputs

## Data Requirements

The notebook expects the following inputs:

- `stage_2_train_labels.csv`
- `stage_2_train_images/` or the corresponding PNG output directory after conversion

During preprocessing and training, the notebook:

- removes duplicate `patientId` entries
- keeps only samples whose PNG files exist on disk
- checks that the CSV and PNG directory are available before training begins

## Preprocessing

A dedicated conversion function reads each DICOM file, extracts the pixel array, normalizes it to 8-bit range, handles `MONOCHROME1` inversion when needed, and saves the result as PNG.

## Experiment Design

### Hold-out experiment

The hold-out pipeline uses a stratified split of:

- Train: 80%
- Validation: 10%
- Test: 10%

The notebook uses a fixed random seed for reproducibility.

### 5-fold cross-validation

The CV pipeline uses:

```python
StratifiedKFold(n_splits=5, shuffle=True, random_state=SEED)
```

The CV section evaluates three configurations:

- Standalone Student
- Proposed Hybrid (ResNet KD)
- Proposed Hybrid (ViT KD)

## Models Found in the Notebook

### Teacher models

The notebook defines two teacher models:

- `TeacherCNN`, based on `torchvision.models.resnet50`
- `TeacherViT`, based on `torchvision.models.vit_b_16`

### Student model

The student model is `HybridStudent`, which uses:

- a small CNN feature extractor
- a Transformer encoder layer
- adaptive average pooling
- a feature-alignment projection layer
- a classifier head

### Baseline models supported in the hold-out section

The notebook includes a baseline model builder for:

- ResNet-50
- ViT-Base
- DeiT-Tiny
- MobileNetV3-Small
- EfficientNet-B0
- MobileViT-XXS

## Training Logic

The knowledge-distillation loss combines:

- cross-entropy loss on labels
- mean-squared-error loss between student and teacher features

The notebook implements this as:

```python
total_loss = alpha * ce_loss + (1 - alpha) * kd_loss
```

The shared notebook uses these key settings:

- `SEED = 42`
- `CV_EPOCHS = 10`
- `BATCH_SIZE = 32`
- `ALPHA = 0.7`

## Metrics

The notebook computes the following metrics:

- Accuracy
- Precision
- Recall
- F1-score
- ROC-AUC

It also generates:

- confusion matrices
- ROC curves
- validation accuracy plots
- fold-wise summary tables

## Output Files

### Hold-out outputs

- `final_test_results.csv`
- `epoch_history.csv`
- `test_predictions_proposed_vitkd.csv`
- `figure2_validation_accuracy.png`
- `figure3_vitkd_testset.png`

### 5-fold CV outputs

- `cv5_results.csv`
- `cv5_history.csv`
- `cv5_summary_mean_std.csv`
- `cv5_paired_ttest_f1.csv`
- `table5_cv_ready.csv`
- `rsna_5fold_outputs.zip`

## Runtime Notes

- The notebook is written for Google Colab.
- It checks whether CUDA is available and selects GPU when possible.
- If `timm` is not installed, the notebook installs it automatically in-place.
- The CV section includes Google Drive backup support and optional local ZIP download.

## Reproducibility

The notebook sets a fixed seed and uses stratified splitting to keep the runs reproducible.

## Dependencies

A matching dependency list is available in `requirements.txt`.

## License

This repository is intended for research and academic use. Please verify the original RSNA/Kaggle dataset terms before redistribution.
