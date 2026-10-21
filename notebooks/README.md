<a id="top"></a>

<p align="center">
  <img src="https://github.com/Kumkumpathare/MammoCare/blob/main/assets/heading-mono-violet.svg" alt="MammoCare Notebooks" width="560">
</p>

<p align="center">
<strong>Aryan Patel</strong> (GitHub: <a href="https://github.com/aryanap07">aryanap07</a>) and <strong>Kumkum Pathare</strong> (GitHub: <a href="https://github.com/Kumkumpathare">Kumkumpathare</a>)
</p>

<p align="center">
Notebooks that classify BreaKHis histopathology images as benign or malignant. Notebook 01 audits the dataset, defines group-separated cross-validation folds and evaluates three baseline models: a majority-class classifier, a small convolutional neural network (CNN) trained from scratch, and a frozen ResNet18.
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
> This is a research workflow. It is not a clinically validated diagnostic tool.

##  • Key findings

- The evaluation splits data by group (never by image) across 5 outer folds.
- Group-level ROC-AUC (mean over 5 folds): ResNet18 (frozen) 0.938, CNN (scratch) 0.917, majority class 0.500.
- With only 81 groups, the difference between the two learned models is small relative to fold-to-fold variation.

##  • Contents

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
## • Introduction

This directory contains the notebooks of MammoCare, a research project on the classification of breast histopathology images as benign or malignant using the BreaKHis dataset. The evaluation is designed so that images from the same group do not appear in both the training and the test data: folds are split by group, never by image.



| Item | Detail |
|---|---|
| Task | Binary classification (benign vs malignant) of BreaKHis histopathology images |
| Evaluation | 5 outer folds, split by group and never by image |
| Models | Majority class, small CNN trained from scratch, ResNet18 (frozen) |
| Hardware | T4 graphics processing unit (GPU) on Kaggle, about 80 minutes for all 5 folds |



---

<a id="notebook-index"></a>
## • Notebook index

| # | Notebook | Contents | Status |
|---|---|---|---|
| 01 | [`01_dataset_audit_grouped_evaluation_baselines.ipynb`](01_dataset_audit_grouped_evaluation_baselines.ipynb) | Dataset audit, group-separated folds, baseline models | Done |
| 02 | [TO BE ADDED] | Augmentation, resizing, fine-tuning | Planned |



---

<a id="repository-structure"></a>
## • Repository structure

- [notebooks/](../notebooks/)
- [assets/](../assets/)
- [docs/](../docs/)
- [outputs/](../outputs/)
- [.github/](../.github/)
- [README.md](../README.md)
- [requirements.txt](../requirements.txt)
- [pyproject.toml](../pyproject.toml)
- [LICENSE](../LICENSE)
- [CONTRIBUTING.md](../CONTRIBUTING.md)
- [SECURITY.md](../SECURITY.md)



---

<a id="dataset-description"></a>
## • Dataset description

| Item | Detail |
|---|---|
| Source | BreaKHis |
| Images | 7,909 (2,480 benign, 5,429 malignant) |
| Magnification | 40X, 100X, 200X, 400X |
| Groups | 81, defined by the full YEAR-SLIDE_ID token in the filename |
| Download size | About 4.3 GB |
| Official dataset page | [BreaKHis dataset archive] (https://www.inf.ufpr.br/vri/databases/BreaKHis_v1.tar.gz)  |

[Back to top](#top)

---

<a id="methodology"></a>
## • Methodology

###  • Pipeline

<p align="center">
  <img src="https://github.com/Kumkumpathare/MammoCare/blob/main/assets/breakhis_workflow.svg" alt="Pipeline: BreaKHis images, manifest and audit checks, group IDs, 5 outer folds, three models, metrics and error review" width="720">
</p>

###  •Models

| Model | Description |
|---|---|
| Majority class | Baseline that always predicts the most frequent class |
| CNN (scratch) | Small convolutional neural network trained from scratch. |
| ResNet18 (frozen) | ResNet18 with frozen layers. |

###  • Evaluation protocol

- Folds are split by group, never by image.
- Image-level predictions are averaged within each group to obtain group-level results.
- The test fold is never used for model selection.

### •Training settings

| Setting | Value |
|---|---|
| Seed | 42 |
| Image size | 224 |
| Batch size | 32 |
| Maximum epochs | 15 |
| Early stopping | Patience of 4 epochs |
| Optimiser | AdamW, learning rate 1e-3, weight decay 1e-4 |
| Loss | Class-weighted cross-entropy |
| Checkpoint selection | Best validation area under the receiver operating characteristic curve (ROC-AUC) |
| Augmentation | None |



---

<a id="installation-and-usage"></a>
## • Installation and usage

###  • Requirements

- A GPU is strongly recommended.
- Python version: Python 3.11 recommended
- Dependencies used by the notebook: torch, torchvision, scikit-learn, pandas, numpy, Pillow, matplotlib, tqdm, ipython and jupyter. The packages pytest and ruff are not required.

###  • Installation

```bash
git clone https://github.com/aryanap07/MammoCare.git
cd MammoCare
pip install -r requirements.txt
```

###  • Usage

Open [`notebooks/01_dataset_audit_grouped_evaluation_baselines.ipynb`](01_dataset_audit_grouped_evaluation_baselines.ipynb) and run it from top to bottom.

For a reduced test run, set `folds_to_run=(0,)` and lower the number of epochs in the Config cell.

###  • Optional settings

<details>
<summary>Environment variables</summary>

| Variable | Purpose | Default |
|---|---|---|
| `BREAKHIS_ROOT` | Use an existing extracted copy and skip the download | not set |
| `BREAKHIS_DATA_DIR` | Download and extraction location (about 4.3 GB) | `./data` |
| `BREAKHIS_URL` | Archive URL | official link |
| `BREAKHIS_OUT` | Output folder | `./outputs` |

</details>



---

<a id="results"></a>
## • Results

Group-level results, mean over 5 folds. Image predictions are averaged within each group.

| Model | Accuracy | Balanced accuracy | ROC-AUC |
|---|---:|---:|---:|
| Majority class | 0.704 | 0.500 | 0.500 |
| CNN (scratch) | 0.852 | 0.863 | 0.917 |
| ResNet18 (frozen) | 0.865 | 0.849 | 0.938 |

<details>
<summary>Image-level results</summary>

| Model | Accuracy | Balanced accuracy | ROC-AUC |
|---|---:|---:|---:|
| Majority class | 0.691 | 0.500 | 0.500 |
| CNN (scratch) | 0.778 | 0.790 | 0.868 |
| ResNet18 (frozen) | 0.812 | 0.814 | 0.892 |

</details>

**Interpretation.**: With only 81 groups, each test fold contains about 16. Fold-to-fold variation is large (group-level balanced accuracy standard deviation: 0.04 for the CNN and 0.12 for ResNet18). The difference between the two models is small relative to this variation, so the results do not establish that either model is superior.

**Error analysis.** : Most errors are false negatives on ductal carcinoma and false positives on adenosis and fibroadenoma.



---

<a id="outputs"></a>
## • Outputs

By default, all generated files are written to the `outputs/` directory. The location can be changed with the `BREAKHIS_OUT` environment variable.

<details>
<summary>List of generated files</summary>

| File | Contents |
|---|---|
| `manifest.csv` | One row per image with parsed metadata and file hash |
| `dataset_audit_checks.csv` | PASS / FAIL / INFO audit results |
| `identifier_collision_review.csv` | Cases sharing a numeric prefix (informational) |
| `duplicate_image_review.csv` | Exact duplicates (none found) |
| `patient_group_mapping_template.csv` | Default case-to-group mapping |
| `group_summary.csv` | Images per group |
| `partitions.csv` | Train / validation / test groups for each fold |
| `results.csv` | Fold-level metrics, image and group level |
| `predictions.csv` | Per-image predicted probabilities |
| `magnification_results.csv` | Metrics for 40X, 100X, 200X and 400X |
| `misclassified_examples.csv` | Images misclassified by the learned models |
| `runs/fold_<k>/<model>/` | `best.pt`, `history.csv`, `run_info.json` |

</details>



---

<a id="limitations"></a>
## • Limitations

- Results come from group-level cross-validation with few groups (81), so they vary considerably between folds.
- This is an academic workflow, not a clinically validated diagnostic tool.
- Groups are derived from filenames and are a conservative proxy for patients, not verified identities.
- Results on BreaKHis alone do not show how the models would perform at other institutions.
- Per-magnification results rest on few groups per fold and should be interpreted with caution.



---

<a id="next-steps"></a>
## • Next steps

Planned as separate, controlled experiments on the same folds:

- Augmentation
- Aspect-ratio-preserving resizing
- ResNet18 fine-tuning
- Magnification-specific analysis
- Stain normalisation
- Grad-CAM for qualitative inspection



---

<a id="citation"></a>
## • Citation

F. Spanhol, L. S. Oliveira, C. Petitjean and L. Heutte, "A Dataset for Breast Cancer Histopathological Image Classification," IEEE Transactions on Biomedical Engineering, 63(7), 1455-1462, 2016.

[Back to top](#top)
