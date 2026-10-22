<a id="top"></a>

<p align="center">
  <img src="https://raw.githubusercontent.com/aryanap07/MammoCare/main/assets/heading-mono-violet.svg" alt="MammoCare Research Notebooks" width="560">
</p>

<p align="center">
  <strong>Aryan Patel</strong> · <a href="https://github.com/aryanap07">aryanap07</a><br>
  <strong>Kumkum Pathare</strong> · <a href="https://github.com/Kumkumpathare">Kumkumpathare</a>
</p>

<p align="center">
  Research notebooks for breast histopathology image analysis and benign-versus-malignant classification using BreaKHis.
</p>

<p align="center">
  <a href="../README.md">Main README</a> ·
  <a href="01_dataset_audit_grouped_evaluation_baselines.ipynb">Notebook 01</a> ·
  <a href="#results">Results</a> ·
  <a href="#installation-and-usage">Installation and usage</a> ·
  <a href="#outputs">Outputs</a> ·
  <a href="#limitations">Limitations</a> ·
  <a href="#citation">Citation</a>
</p>

> [!IMPORTANT]
> This is an academic research workflow, not a clinically validated diagnostic tool. Its outputs must not be used for medical screening, diagnosis, or treatment decisions.

## Key findings

- Notebook 01 audits the BreaKHis dataset, assigns filename-derived groups, and evaluates models across five outer folds. The partitions are separated by group, never by individual image.
- Mean group-level ROC-AUC across the five outer folds was **0.938** for frozen ResNet18, **0.917** for the CNN trained from scratch, and **0.500** for the majority-class baseline.
- The dataset has only 81 default groups, and fold-to-fold variability is meaningful. These results do not establish that either learned model is superior.

## Contents

1. [Introduction](#introduction)
2. [Notebook index](#notebook-index)
3. [Repository structure](#repository-structure)
4. [Dataset description](#dataset-description)
5. [Methodology](#methodology)
6. [Installation and usage](#installation-and-usage)
7. [Results](#results)
8. [Outputs](#outputs)
9. [Limitations](#limitations)
10. [Next steps](#next-steps)
11. [Citation](#citation)

---

<a id="introduction"></a>
## Introduction

This directory contains the MammoCare notebooks for breast histopathology image classification using the BreaKHis dataset. Notebook 01 establishes a reproducible baseline workflow covering dataset audit, group assignment, cross-validation, model training, metrics, and error review.

| Item | Detail |
|---|---|
| Task | Binary classification: benign versus malignant |
| Dataset | BreaKHis |
| Evaluation | Five outer folds separated by filename-derived group |
| Models | Majority-class baseline, CNN trained from scratch, frozen ResNet18 |
| Recorded environment | Kaggle with a T4 GPU; runtime will vary with hardware and environment |

<a id="notebook-index"></a>
## Notebook index

| # | Notebook | Contents | Status |
|---|---|---|---|
| 01 | [`01_dataset_audit_grouped_evaluation_baselines.ipynb`](01_dataset_audit_grouped_evaluation_baselines.ipynb) | Dataset audit, group-separated folds, baseline models, evaluation and error review | Complete for the current baseline stage |
| 02 | Planned | Controlled experiments including augmentation and ResNet18 fine-tuning | Planned |

<a id="repository-structure"></a>
## Repository structure

```text
MammoCare/
├── assets/
│   ├── heading-mono-violet.svg
│   └── breakhis_workflow.svg
├── notebooks/
│   ├── README.md
│   └── 01_dataset_audit_grouped_evaluation_baselines.ipynb
├── outputs/                 # Generated locally; not required to be committed
├── pyproject.toml
└── README.md
```

The repository may contain additional project files. The notebook writes generated artifacts to `outputs/` by default.

<a id="dataset-description"></a>
## Dataset description

| Item | Detail |
|---|---|
| Source | [Official BreaKHis dataset page](https://web.inf.ufpr.br/vri/databases/breast-cancer-histopathological-database-breakhis/) |
| Images | 7,909 total: 2,480 benign and 5,429 malignant |
| Magnifications | 40×, 100×, 200×, 400× |
| Default groups | 81 filename-derived groups |
| Approximate archive size | 4.3 GB; actual download and extracted size may vary |
| Direct archive used by the notebook | [BreaKHis archive](http://www.inf.ufpr.br/vri/databases/BreaKHis_v1.tar.gz) |

The notebook uses the complete `YEAR-SLIDE_ID` filename token for grouping. Alphabetic suffixes are preserved; the numeric prefix is used only for collision review. Where the same complete token appears across diagnostic subtype folders, those records are conservatively assigned to the same group. These filename-derived groups are proxies, not verified clinical patient identities. Review the generated group mapping and audit reports before interpreting group-level metrics.

[Back to top](#top)

<a id="methodology"></a>
## Methodology

### Pipeline

<p align="center">
  <img src="https://raw.githubusercontent.com/aryanap07/MammoCare/main/assets/breakhis_workflow.svg" alt="BreaKHis workflow: dataset audit, group assignment, five outer folds, baseline models, metrics and error review" width="720">
</p>

### Models

| Model | Description |
|---|---|
| Majority class | Baseline that predicts the most frequent class in the training data |
| CNN (scratch) | Small convolutional neural network trained from scratch |
| ResNet18 (frozen) | ResNet18 feature extractor with frozen pretrained layers and a trained classification head |

### Evaluation protocol

- The notebook creates five outer folds over the filename-derived groups. Groups assigned to train, validation, and test are kept separate within each fold.
- Validation ROC-AUC is used for neural-model checkpoint selection. The test partition is not used for model selection.
- Image-level probabilities are averaged within each group for group-level reporting.
- The default grouping can be overridden with a documented `outputs/patient_group_mapping.csv` file, but a mapping must not split records that share the same full filename token.

### Baseline training settings

| Setting | Value |
|---|---|
| Random seed | 42 |
| Input size | 224 × 224 pixels |
| Batch size | 32 |
| Maximum epochs | 15 |
| Early stopping patience | 4 epochs |
| Optimizer | AdamW |
| Learning rate | 1e-3 |
| Weight decay | 1e-4 |
| Loss | Class-weighted cross-entropy |
| Checkpoint selection | Highest validation ROC-AUC |
| Image normalization | ImageNet mean and standard deviation |
| Data augmentation | None in the baseline experiment |

<a id="installation-and-usage"></a>
## Installation and usage

### Requirements

- Python **3.12 or newer**, consistent with `pyproject.toml`.
- PyTorch, torchvision, scikit-learn, pandas, NumPy, Pillow, matplotlib, and tqdm.
- Jupyter for interactive notebook execution.
- A CUDA-capable GPU is recommended for the full five-fold run. The notebook falls back to CPU when CUDA is unavailable, but execution may take longer.

### Installation

From the repository root:

```bash
git clone https://github.com/aryanap07/MammoCare.git
cd MammoCare
python -m pip install -e ".[dev]"
```

The `.[dev]` extra installs the project dependencies together with the development tools declared in `pyproject.toml`, including Jupyter.

### Run the notebook

Open [`01_dataset_audit_grouped_evaluation_baselines.ipynb`](01_dataset_audit_grouped_evaluation_baselines.ipynb) from the `notebooks/` directory and execute the cells from top to bottom. The first run downloads and extracts BreaKHis unless `BREAKHIS_ROOT` points to an existing extracted dataset.

For a quick pipeline check, set `folds_to_run=(0,)` in the configuration cell and reduce the epoch count. Treat results from a reduced run as a smoke test, not as the five-fold experiment's results.

### Optional environment variables

| Variable | Purpose | Default |
|---|---|---|
| `BREAKHIS_ROOT` | Path to an existing extracted dataset; skips the download when valid | Not set |
| `BREAKHIS_DATA_DIR` | Download and extraction directory | `./data` |
| `BREAKHIS_URL` | Dataset archive URL | Notebook's BreaKHis archive URL |
| `BREAKHIS_OUT` | Directory for generated outputs | `./outputs` |

<a id="results"></a>
## Results

The tables below report the mean across the five executed outer folds. Group-level predictions are formed by averaging image probabilities within each filename-derived group. Values are rounded to three decimal places and reflect the saved notebook run, not a guarantee of future performance.

### Group-level metrics

| Model | Accuracy | Balanced accuracy | ROC-AUC |
|---|---:|---:|---:|
| Majority class | 0.704 | 0.500 | 0.500 |
| CNN (scratch) | 0.852 | 0.863 | 0.917 |
| ResNet18 (frozen) | 0.865 | 0.849 | 0.938 |

### Image-level metrics

| Model | Accuracy | Balanced accuracy | ROC-AUC |
|---|---:|---:|---:|
| Majority class | 0.691 | 0.500 | 0.500 |
| CNN (scratch) | 0.778 | 0.790 | 0.868 |
| ResNet18 (frozen) | 0.812 | 0.814 | 0.892 |

### Interpretation and error review

Group-level balanced-accuracy standard deviation across folds was **0.040** for the CNN and **0.122** for frozen ResNet18. With only 81 groups and this degree of fold-to-fold variation, the mean scores do not establish that one learned model is superior.

The saved error review includes recurring false negatives among ductal carcinoma images and false positives among adenosis and fibroadenoma images. These are error-analysis observations from this run, not clinical conclusions. Review the fold-level and per-magnification counts before drawing further conclusions.

<a id="outputs"></a>
## Outputs

Generated artifacts are written to `outputs/` by default. Set `BREAKHIS_OUT` to change the location.

| File | Contents |
|---|---|
| `manifest.csv` | One row per image with parsed metadata and file hash |
| `dataset_audit_checks.csv` | PASS / FAIL / INFO audit results |
| `identifier_collision_review.csv` | Cases sharing a numeric prefix; for review only |
| `duplicate_image_review.csv` | Exact-duplicate image review |
| `patient_group_mapping_template.csv` | Template for a reviewed case-to-group mapping |
| `group_summary.csv` | Image counts per group |
| `partitions.csv` | Train, validation, and test group assignments by fold |
| `results.csv` | Fold-level image- and group-level metrics |
| `predictions.csv` | Per-image predictions and probabilities |
| `magnification_results.csv` | Metrics by magnification and evaluation level |
| `misclassified_examples.csv` | Misclassified image examples for error review |
| `runs/fold_<k>/<model>/best.pt` | Selected model checkpoint |
| `runs/fold_<k>/<model>/history.csv` | Training history |
| `runs/fold_<k>/<model>/run_info.json` | Recorded run configuration and metadata |

Keep the audit reports, grouping policy, and run configuration with any results shared outside the project. The group-level estimates depend on the reviewed grouping map.

<a id="limitations"></a>
## Limitations

- The default groups are derived from filenames and are not verified patient identities.
- Only 81 default groups are available; fold-level variation is substantial for some metrics.
- BreaKHis-only results do not establish performance on other datasets, institutions, staining protocols, or acquisition systems.
- Per-magnification metrics involve smaller subsets and should be interpreted cautiously.
- No external clinical validation has been established. The notebook is intended for research and education, not clinical use.

<a id="next-steps"></a>
## Next steps

Planned as controlled experiments using the same reviewed group assignments where appropriate:

- Evaluate data augmentation.
- Compare aspect-ratio-preserving resizing with the baseline resize.
- Fine-tune ResNet18.
- Investigate magnification-specific performance.
- Evaluate stain normalization.
- Use Grad-CAM for qualitative model inspection, with appropriate interpretive caution.

<a id="citation"></a>
## Citation

Spanhol, F., Oliveira, L. S., Petitjean, C., and Heutte, L. “A Dataset for Breast Cancer Histopathological Image Classification.” *IEEE Transactions on Biomedical Engineering*, 63(7), 1455–1462, 2016.

[Back to top](#top)
