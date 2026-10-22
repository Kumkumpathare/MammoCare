<p align="center">
  <img src="https://raw.githubusercontent.com/aryanap07/MammoCare/main/assets/mammocare-logo.svg" alt="MammoCare" width="460">
</p>

<h1 align="center">Research Notebooks</h1>

<p align="center">
  <strong>A reproducible notebook workflow for breast histopathology image analysis.</strong>
</p>

<p align="center">
  <a href="../README.md">Main project</a> ·
  <a href="01_dataset_audit_grouped_evaluation_baselines.ipynb">Notebook 01</a> ·
  <a href="#results">Results</a> ·
  <a href="#getting-started">Getting started</a> ·
  <a href="#outputs">Outputs</a> ·
  <a href="#limitations">Limitations</a>
</p>

---

## About this notebook collection

MammoCare's notebooks provide a transparent, step-by-step research workflow for studying breast histopathology images with data science and deep learning. Each experiment is intended to be inspectable and reproducible: data preparation is recorded, evaluation groups are defined before model training, and results are reported alongside the limitations that affect their interpretation.

**Notebook 01** establishes the foundation. It audits the BreaKHis dataset, creates group-separated evaluation folds, and compares three reference models for classifying images as **benign** or **malignant**.

> **Research scope:** This repository is for academic research. The models and results are not clinically validated and must not be used for screening, diagnosis, treatment, or other clinical decisions.

## Notebook index

| Notebook | Purpose | Status |
|---|---|---|
| [`01_dataset_audit_grouped_evaluation_baselines.ipynb`](01_dataset_audit_grouped_evaluation_baselines.ipynb) | Dataset audit, filename-derived grouping, five-fold evaluation, baseline models, and error review | Complete baseline study |
| Notebook 02 | Controlled experiments with augmentation, resizing, and model fine-tuning | Planned |

## Notebook 01 at a glance

| Item | Description |
|---|---|
| Research task | Binary classification of BreaKHis histopathology images: benign or malignant |
| Dataset | 7,909 images across four magnifications |
| Evaluation | Five outer folds with training, validation, and test partitions separated by group |
| Models | Majority-class baseline, CNN trained from scratch, frozen ImageNet-pretrained ResNet18 |
| Primary reporting unit | Group-level metrics, with image-level metrics reported for comparison |
| Main deliverables | Data manifest, audit reports, fold assignments, model histories, checkpoints, predictions, and metrics |

## Dataset

The study uses **BreaKHis**, a public breast cancer histopathology image dataset. The notebook expects the original image archive or an extracted copy; the dataset itself is not included in this repository.

- **Images:** 7,909
- **Benign:** 2,480
- **Malignant:** 5,429
- **Magnifications:** 40×, 100×, 200×, and 400×
- **Filename-derived diagnostic case IDs:** 82
- **Default grouping IDs:** 81

The class distribution is uneven, so accuracy alone is not enough to describe model performance. Balanced accuracy, sensitivity, specificity, and ROC-AUC are reported as well.

**Dataset source:** [Official BreaKHis database page](https://web.inf.ufpr.br/vri/databases/breast-cancer-histopathological-database-breakhis/). Please consult the source for dataset terms and citation guidance.

## Methodology

<p align="center">
  <img src="https://raw.githubusercontent.com/aryanap07/MammoCare/main/assets/breakhis_workflow.svg" alt="BreaKHis workflow: image audit, filename-derived groups, group-separated folds, baseline models, metrics, and error review" width="760">
</p>

### 1. Audit the data before modelling

The notebook builds a manifest with parsed labels, subtype, magnification, dimensions, file hashes, and filename-derived identifiers. It checks filename parsing, folder-label agreement, image readability, expected magnifications, missing metadata, and exact duplicate hashes. Files that cannot be parsed or read remain visible in the audit rather than being silently omitted.

The recorded run found 7,909 images, no unparsed filenames, no unreadable images, no exact-duplicate image hashes, and no filename-to-folder mismatches. Identifier collisions are reported separately as review information; they are not automatically treated as data-integrity failures.

### 2. Define groups carefully

The default grouping key is the complete `YEAR-SLIDE_ID` token from each filename, including any alphabetic suffix. A shortened numeric prefix is retained for audit purposes only; it is **not** used as a grouping key.

One complete token, `14-13412`, appears under both the ductal carcinoma (DC) and lobular carcinoma (LC) subtype folders. The notebook conservatively assigns these records to the same split group to reduce the risk of cross-partition leakage, while preserving their separate diagnostic case records in the manifest. This yields 82 filename-derived diagnostic case IDs and 81 default grouping IDs.

These identifiers are **filename-derived grouping proxies, not verified clinical patient identities**. If an authoritative and documented patient mapping becomes available, use it to review or refine the grouping policy without splitting records that share a complete token.

### 3. Create group-separated evaluation folds

The notebook creates five outer folds from a table containing one row per grouping ID, rather than splitting individual images. In each outer fold, the test groups are held out; the remaining groups are split again to form training and validation partitions. A group must not appear in more than one partition within a fold, and each group is used as test data once across the five outer folds.

Model selection is based on validation ROC-AUC. The test partition is not used to choose checkpoints or tune settings. Image probabilities are averaged within each group to produce the group-level predictions used for the primary comparison.

### 4. Compare three baseline models

| Model | Role in the study |
|---|---|
| Majority-class baseline | Predicts the most frequent class in the training partition. It establishes a simple reference point. |
| CNN trained from scratch | Provides a compact learned baseline without pretrained image features. |
| Frozen ResNet18 | Uses an ImageNet-pretrained ResNet18 backbone with a newly trained classification head; the backbone remains frozen. |

Fine-tuning ResNet18 and applying augmentation are left for later controlled experiments, so their effects can be assessed separately.

### 5. Training configuration

| Setting | Value |
|---|---|
| Random seed | 42 |
| Input size | 224 × 224 pixels |
| Batch size | 32 |
| Maximum epochs | 15 |
| Early-stopping patience | 4 epochs |
| Optimiser | AdamW |
| Learning rate | 0.001 |
| Weight decay | 0.0001 |
| Loss | Class-weighted cross-entropy |
| Checkpoint selection | Best validation ROC-AUC |
| Image normalisation | ImageNet mean and standard deviation |
| Augmentation | None in this baseline |
| Classification threshold | 0.5 |

Images are resized directly to a square input in this baseline. Aspect-ratio-preserving resizing and data augmentation are reserved for follow-up experiments.

## Results

The table below reports the means across the five outer folds saved in the notebook. **Group-level results are the primary comparison** because images from one group can be correlated. Image-level scores are included to show how the evaluation unit affects the reported result.

| Model | Image-level accuracy | Image-level ROC-AUC | Group-level accuracy | Group-level balanced accuracy | Group-level ROC-AUC |
|---|---:|---:|---:|---:|---:|
| Majority-class baseline | 0.691 | 0.500 | 0.704 | 0.500 | 0.500 |
| CNN trained from scratch | 0.778 | 0.868 | 0.852 | 0.863 | 0.917 |
| Frozen ResNet18 | 0.812 | 0.892 | 0.865 | 0.849 | 0.938 |

### How to read these results

- Both learned models outperform the majority-class baseline on ROC-AUC in this run.
- Frozen ResNet18 has the higher mean group-level ROC-AUC, while the CNN has the higher mean group-level balanced accuracy.
- Scores vary across folds. Group-level ROC-AUC standard deviation is 0.091 for the CNN and 0.069 for frozen ResNet18; group-level balanced-accuracy standard deviation is 0.040 and 0.122, respectively.
- These averages do not establish that one learned model is generally superior. There are only 81 default grouping IDs, and results depend on the filename-based grouping assumption.

For a complete view, consult the notebook's fold-level metrics, sensitivity, specificity, precision, malignant-class F1-score, confusion matrices, magnification-specific results, and misclassified examples. Magnification-specific metrics should be interpreted cautiously because each fold contains relatively few groups.

## Evaluation metrics

The positive class is **malignant**. The notebook reports the following measures at image and group level:

| Metric | What it tells you |
|---|---|
| Accuracy | Fraction of predictions that are correct overall |
| Balanced accuracy | Average recall across the benign and malignant classes; useful when class counts differ |
| Sensitivity | Fraction of malignant examples correctly identified |
| Specificity | Fraction of benign examples correctly identified |
| Precision | Fraction of examples predicted malignant that are truly malignant in the dataset labels |
| Malignant F1-score | Balance between malignant-class precision and sensitivity |
| ROC-AUC | How well predicted probabilities rank malignant examples above benign examples across thresholds |

No single metric captures every aspect of performance. Results should be read together, with fold variation and the evaluation unit kept in view.

## Getting started

### Requirements

Use Python 3.12 or newer, with the packages listed by the project in `requirements.txt`. The notebook uses PyTorch, torchvision, scikit-learn, pandas, NumPy, Pillow, matplotlib, and tqdm. A GPU is recommended for the full five-fold run; a CPU can be used for a smaller pipeline check but will be slower.

### Install the project dependencies

From the repository root:

```bash
git clone https://github.com/aryanap07/MammoCare.git
cd MammoCare
python -m venv .venv
```

Activate the environment, then install the project requirements.

**Windows PowerShell**

```powershell
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

**macOS or Linux**

```bash
source .venv/bin/activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### Open the notebook

Open `notebooks/01_dataset_audit_grouped_evaluation_baselines.ipynb` in Jupyter or your preferred notebook environment, then run the cells from top to bottom. Google Colab or Kaggle with a GPU is recommended for the full baseline comparison. The first ResNet18 initialisation may download pretrained ImageNet weights.

The default configuration runs all five outer folds. To check that the pipeline executes before committing to the full run, set `cfg.folds_to_run = (0,)` and reduce `cfg.epochs` in the configuration cell. A single-fold smoke run is for debugging; it is not a substitute for the five-fold results reported above.

### Dataset location

The notebook can download the archive when no local dataset is found. If you already have an extracted copy, set `BREAKHIS_ROOT` to the directory containing the extracted BreaKHis tree.

| Environment variable | Purpose | Default or behaviour |
|---|---|---|
| `BREAKHIS_ROOT` | Path to an existing extracted dataset | Not set; the notebook looks for data and may download the archive |
| `BREAKHIS_DATA_DIR` | Download and extraction directory | `./data` |
| `BREAKHIS_URL` | Dataset archive URL | Set by the notebook to the BreaKHis archive URL |
| `BREAKHIS_OUT` | Directory for generated outputs | `./outputs` |

The download is approximately 4.27 GB. Ensure the runtime has enough storage before starting.

## Outputs

Generated files are written to `outputs/` unless `BREAKHIS_OUT` points elsewhere.

| File or directory | Purpose |
|---|---|
| `manifest.csv` | One record per discovered image, including parsed metadata, file hash, and audit fields |
| `dataset_audit_checks.csv` | Dataset integrity checks and their statuses |
| `identifier_collision_review.csv` | Identifier-prefix collisions for review; prefixes are not used as grouping keys |
| `duplicate_image_review.csv` | Exact duplicate-image hash review |
| `patient_group_mapping_template.csv` | Template for a documented, manually reviewed group mapping |
| `group_summary.csv` | Image and label summaries for each grouping ID |
| `partitions.csv` | Training, validation, and test group assignments for every outer fold |
| `results.csv` | Fold-level evaluation metrics at image and group level |
| `predictions.csv` | Per-image predictions and probabilities |
| `magnification_results.csv` | Evaluation metrics by magnification, with image and group counts |
| `misclassified_examples.csv` | False-positive and false-negative examples from the learned models |
| `runs/fold_<k>/<model>/` | Model checkpoint (`best.pt`), training history (`history.csv`), and run configuration (`run_info.json`) |

Keep these files with experiment notes when reporting a result. The manifest, grouping policy, partition assignments, and run configuration are essential for interpreting and reproducing the comparison.

## Limitations

- **Grouping is not verified patient identity.** The filenames do not provide an authoritative clinical patient metadata table. The full-token policy is a cautious proxy, and its limitations must be stated in any report.
- **The number of groups is small.** With 81 default grouping IDs, fold-level performance can vary substantially. Mean scores should always be read alongside variability.
- **The evaluation is internal to BreaKHis.** These results do not establish performance on images from other institutions, scanners, staining protocols, or patient populations.
- **Subgroup counts are limited.** Per-magnification results may be based on few groups and should not be over-interpreted.
- **The workflow is not clinically validated.** No result in this repository should be treated as evidence of clinical readiness.

## Next steps

The next experiments should change one major factor at a time while keeping the reviewed group partitions fixed wherever possible:

1. Review fold-to-fold variation and inspect false positives and false negatives.
2. Compare augmentation strategies against the unaugmented baseline.
3. Evaluate aspect-ratio-preserving resizing.
4. Fine-tune some or all ResNet18 layers under a documented protocol.
5. Investigate performance by magnification, with group counts shown alongside each metric.
6. Evaluate stain normalisation as a separate preprocessing experiment.
7. If Grad-CAM is added, use it only for qualitative inspection; visual explanations do not prove that a model has learned medically meaningful features.

## Citation

If you use BreaKHis, cite the original dataset paper and consult the official source for its terms and citation guidance:

F. Spanhol, L. S. Oliveira, C. Petitjean, and L. Heutte, “A Dataset for Breast Cancer Histopathological Image Classification,” *IEEE Transactions on Biomedical Engineering*, 63(7), 1455–1462, 2016.

[Official BreaKHis database page](https://web.inf.ufpr.br/vri/databases/breast-cancer-histopathological-database-breakhis/)

---

<p align="center">
  <sub>MammoCare · Research that is documented, reproducible, and interpreted with care.</sub>
</p>
