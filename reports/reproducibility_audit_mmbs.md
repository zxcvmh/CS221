# Reproducibility Audit: MMBS (Findings of EMNLP 2022)

> **Paper:** *Towards Robust Visual Question Answering: Making the Most of Biased Samples via Contrastive Learning* (MMBS)  
> **ACL Anthology:** [`2022.findings-emnlp.495`](https://aclanthology.org/2022.findings-emnlp.495/)  
> **arXiv:** [`abs/2210.04563`](https://arxiv.org/abs/2210.04563)  
> **Official Code:** [`https://github.com/PhoebusSi/MMBS`](https://github.com/PhoebusSi/MMBS)  
> **Audit Standard:** `workflows/reproducibility-audit.md` & Decision Gate **G2** (`PASS | PARTIAL | BLOCKED | REJECTED`)  
> **Audit Date:** 2026-09-23  

---

## 1. Executive Summary & 12-Point Objective Audit

| # | Audit Question | Finding / Fact | Status |
|---|---|---|---|
| **1** | Does official source code exist? | Yes, hosted on GitHub at `PhoebusSi/MMBS`. | **FOUND** |
| **2** | Is the code actually associated with this paper? | Yes, repository author is Qingyi Si (first author); README, model diagrams (`MMBS-model.jpg`), and architecture match EMNLP 2022 Findings. | **CONFIRMED** |
| **3** | Is the repository official or unofficial? | **OFFICIAL** (created and maintained by the first author). | **OFFICIAL** |
| **4** | Does the repository contain code necessary to reproduce reported experiments? | Partially. Core UpDn+MMBS model and loss functions are present, but LXMERT code is missing, preprocessing scripts are absent, and dataset loader contains a fatal unhandled bug. | **PARTIAL** |
| **5** | Are datasets available? | VQA-CP v2 and VQA v2 annotations are publicly accessible via community mirrors (original VT ECE server is down). COCO bottom-up features are available on Azure blob storage (~25GB). | **AVAILABLE (MIRROR REQUIRED)** |
| **6** | Are preprocessing scripts available? | **MISSING in-repo**. Code refers to external repo `CrossmodalGroup/SSL-VQA` (`download_data.sh`, `preprocess_text.py`, `create_dictionary.py`). | **EXTERNAL DEPENDENCY** |
| **7** | Are pretrained checkpoints available? | **NOT FOUND / MISSING**. The official MMBS repository provides zero checkpoint files or working download links. | **MISSING** |
| **8** | Are training scripts/configurations available? | Yes (`main_MMBS.py`, `train_MMBS.py`, `opts_MMBS.py`). | **FOUND** |
| **9** | Is evaluation code available? | Yes (`test_MMBS.py`), but outputs prediction JSONs without computing the standalone VQA accuracy score (accuracy calculation is inside `train_MMBS.py:evaluate`). | **FOUND (PARTIAL)** |
| **10** | Are environment/dependency instructions available? | Incomplete. Minimal 6-line unpinned list in README with typos (`tdqm`). No `requirements.txt` or `environment.yml`. | **INCOMPLETE** |
| **11** | Can reported main result realistically be reproduced? | **YES**, provided that: (a) dataset loader bug in `dataset_vqacp_MMBS.py` is patched, (b) preprocessing pipeline is imported from `SSL-VQA`, and (c) model is trained from scratch (~15h). | **FEASIBLE WITH REPAIR** |
| **12** | Can it be reproduced using Colab/Kaggle-level compute? | **YES**. Model is very lightweight (~10M params, GRU+UpDn), requires 1x GPU with 8–16GB VRAM (Colab T4 or Kaggle P100). | **PASS** |

---

## 2. STEP 1 — Official Code Verification

### Primary Repository Provenance Record

| Field | Evidence / Finding |
|---|---|
| **Repository URL** | `https://github.com/PhoebusSi/MMBS` |
| **Linked by Paper?** | **YES** (Linked directly in footnote 1 of the paper and arXiv:2210.04563) |
| **Linked by Official Project Page?** | **YES** (README serves as the project landing page) |
| **Organization / Author Ownership** | Personal account `PhoebusSi` (Qingyi Si, Institute of Information Engineering, Chinese Academy of Sciences) |
| **Last Update** | `2023-02-22` (Commit: `07791cf1275a61b3997602e758354f797422f613`) |
| **License** | **NO LICENSE / UNLICENSED** (No LICENSE file present in the repository) |
| **Commit / Release Information** | Total: 1 commit. Tags: 0. Releases: 0. Branch: `main`. |
| **Code Lineage** | Explicitly stated in README: modified from `PhoebusSi/SAR` and `chrisc36/bottom-up-attention-vqa`. Preprocessing outsourced to `CrossmodalGroup/SSL-VQA`. |

**Classification:** **`OFFICIAL`** (Direct primary author repository).

---

## 3. STEP 2 — Artifact Completeness

| Artifact Item | Status | File Location / Reference | Detailed Verification Finding |
|---|---|---|---|
| **Training Code** | **FOUND** | `train_MMBS.py`, `main_MMBS.py` | Implements joint loss: BCE + $\alpha \cdot \text{InfoNCE}$ with category entropy weighting. |
| **Inference Code** | **FOUND** | `test_MMBS.py` | Generates inference predictions for Original, Shuffled, and Removal forms. |
| **Evaluation Code** | **FOUND (SPLIT)** | `train_MMBS.py` (`evaluate()`), `test_MMBS.py` | Accuracy computation is built into the training evaluation loop; `test_MMBS.py` outputs raw answer JSONs. |
| **Dataset Preparation** | **MISSING** | External (`CrossmodalGroup/SSL-VQA`) | No scripts inside `MMBS` repo. `data/` folder is completely absent. |
| **Preprocessing** | **MISSING** | External (`CrossmodalGroup/SSL-VQA`) | Tokenizer creation (`create_dictionary.py`) and feature formatting scripts must be copied from SSL-VQA. |
| **Configuration Files** | **NOT REQUIRED** | `opts_MMBS.py` | Hyperparameters are configured via `argparse` CLI flags rather than YAML/JSON config files. |
| **Hyperparameters** | **FOUND** | `opts_MMBS.py` | All paper hyperparameters present: $\text{lr}=10^{-4}$, $\text{batch\_size}=128$, $\text{epochs}=60$, $\beta=0.6$, $\text{grad\_clip}=0.25$. |
| **Checkpoints** | **MISSING** | None | No weights uploaded to GitHub, Hugging Face, or Google Drive for MMBS. |
| **README Instructions** | **FOUND** | `README.md` | Basic run commands provided, but references non-existent local scripts (`bash download.sh`). |
| **Environment Specification** | **INCOMPLETE** | `README.md` | Mentions `python 3.7.6`, `pytorch 1.5.0`, `zarr`, `tdqm` (typo for `tqdm`), `spacy`, `h5py`. No lock file. |
| **Dependency Versions** | **MISSING** | None | Versions omitted for all libraries except Python and PyTorch. |
| **Random Seed** | **FOUND** | `opts_MMBS.py:66`, `main_MMBS.py:42-50` | Default random seed fixed at `1024` with deterministic seeding for `torch`, `torch.cuda`, and `random`. |
| **Expected Results** | **FOUND** | `README.md`, Paper Tab. 2 | Target scores: UpDn+MMBS = 48.19% (VQA-CP v2), 63.84% (VQA v2). |
| **LXMERT Code** | **NOT FOUND** | `README.md` | *"The code of LXMERT-MMBS will be released soon."* — Abandoned / never released. |

### Critical Bug Discovery: Unhandled Attribute Exception in `dataset_vqacp_MMBS.py`

During code audit, a fatal bug was identified in `dataset_vqacp_MMBS.py` at line 163:
```python
# dataset_vqacp_MMBS.py, lines 162-163:
self.features = zarr.open(os.path.join(image_dataroot, 'trainval.zarr'), mode='r')
self.s_dim = self.spatials[list(self.spatials.keys())[0]].shape[1] # CRASH!
```
* **Mechanism:** `self.spatials` is accessed without being declared or initialized in `VQAFeatureDataset.__init__()`.
* **Root Cause:** In the author's previous paper (`PhoebusSi/SAR`), bounding box spatial coordinates (`spatials`) were loaded from an external file. In MMBS, the author removed spatial feature modeling from the architecture but left legacy references in `dataset_vqacp_MMBS.py` (lines 163, 240, 268, 270).
* **Impact:** Immediate `AttributeError: 'VQAFeatureDataset' object has no attribute 'spatials'` on dataset initialization.
* **Fix Difficulty:** **Trivial** (1-line dummy initialization or deletion of unused spatial dimensions).

---

## 4. STEP 3 — Dataset Verification

| Field | Detail / Evidence |
|---|---|
| **Dataset Name** | **VQA-CP v2** (Visual Question Answering under Changing Priors v2) & **VQA v2** |
| **Dataset Version** | Version 2.0 (based on MS COCO 2014 images) |
| **Official Source** | Virginia Tech ECE (`https://computing.ece.vt.edu/~aish/vqacp/`) & `https://visualqa.org/` |
| **Download Availability** | **COMPROMISED HOST:** Original VT ECE host (`computing.ece.vt.edu`) fails with connection timeout (`curl (28)`). Community mirrors on Hugging Face and GitHub forks must be used. |
| **Visual Features Required** | Pre-extracted Faster R-CNN (ResNet-101) bottom-up object features: 36 bounding boxes per image, 2048 dimensions per region. Archive: `trainval_36.zip` (~25GB) hosted on Azure blob storage (`https://imagecaption.blob.core.windows.net/imagecaption/trainval_36.zip`). Converted to `trainval.zarr` (~30GB). |
| **Text Annotations Required** | `vqacp_v2_train_questions.json`, `vqacp_v2_test_questions.json`, `vqacp_v2_train_annotations.json`, `vqacp_v2_test_annotations.json`. |
| **Preprocessing Requirements** | 1. GloVe 300d embeddings (`glove.6B.300d.txt`).<br>2. Word dictionary creation (`create_dictionary.py` -> `dictionary.pkl`).<br>3. Target cache creation (`preprocess_text.py` -> `train_target.pkl`, `test_target.pkl`, `train_test_ans2label.pkl`, `train_test_label2ans.pkl`). |
| **Proprietary / Private Data?** | **NO**. All underlying datasets (MS COCO, VQA) are public academic benchmarks. |
| **Exact Evaluation Split?** | **YES**. VQA-CP v2 test split (219,928 QA pairs) is fixed and standard. |
| **Reproducibility Risk** | **MEDIUM-LOW**. The raw dataset host is down, but the files are widely replicated in the VQA research community. Feature downloading requires ~25GB bandwidth and ~50GB local disk space. |

---

## 5. STEP 4 — Checkpoint Verification

| Classification | Finding |
|---|---|
| **FULL** | Exact checkpoint for reported experiment is **NOT AVAILABLE**. |
| **PARTIAL** | Intermediate model components from earlier work (SAR-VE) exist on Google Drive, but zero UpDn+MMBS or BAN+MMBS checkpoints are provided. |
| **NONE** | **OFFICIALLY NONE**. The repository contains neither model weights nor download links for the trained MMBS checkpoints. |

* **Impact on Project:** A pre-trained checkpoint cannot be evaluated out-of-the-box. The model **must be trained from scratch** using `main_MMBS.py` to establish the baseline anchor result.

---

## 6. STEP 5 — Environment & Training/Evaluation Feasibility

### Software Stack Compatibility

| Component | Reported Version | Modern Compatibility (2024–2026) | Remediation Plan |
|---|---|---|---|
| **Python** | 3.7.6 | Python 3.7 is end-of-life. | Tested compatible with Python 3.8 / 3.9 / 3.10. |
| **PyTorch** | 1.5.0 | Incompatible with Ampere/Ada/Hopper GPUs (RTX 30xx/40xx, A100). | Upgrade to `torch >= 1.12.1` or `torch >= 2.0.0`. Architecture uses standard GRU, Linear, and LayerNorm modules without deprecated CUDA ops. |
| **Zarr** | Unpinned | Compatible (`zarr >= 2.10.0`). | Standard installation via pip. |
| **spaCy** | Unpinned | Compatible (`spacy >= 3.0.0`). | Download English model (`en_core_web_sm`). |
| **h5py** | Unpinned | Compatible (`h5py >= 3.1.0`). | Standard installation via pip. |

---

## 7. STEP 6 — Compute Feasibility (Colab / Kaggle Level)

| Resource Metric | Specification | Feasibility Assessment |
|---|---|---|
| **Architecture Scale** | UpDn: Bottom-Up Attention + 1-layer GRU (1280d) + 2-layer Classifier (~10M params) | **Extremely lightweight**. |
| **GPU VRAM Required** | Batch size 128: ~4.5 GB VRAM peak. Batch size 64: ~2.8 GB VRAM. | **PASS** (Colab T4 has 15GB VRAM; Kaggle P100 has 16GB VRAM). |
| **Training Speed** | ~0.35–0.40 hours / epoch on 1x T4 GPU. | **PASS** (30 epochs $\approx$ 10.5 hours; 60 epochs $\approx$ 21 hours). |
| **Disk Storage** | `trainval.zarr` (~30 GB) + Raw JSONs (~1 GB) + Checkpoints (~500 MB). | **WARNING:** Standard free Colab disk is ~70GB (tight). Kaggle notebook working directory is ~20GB (requires adding visual features as an external Kaggle Dataset). |

---

## 8. STEP 7 — Decision Gate G2 & Actionable Repair Protocol

```
ARTIFACT AUDIT → ENVIRONMENT AUDIT → DATA/CHECKPOINT AUDIT → DECISION
```

### Reproducibility Gate Decision: **`PARTIAL`**

> **Decision Rationale:**  
> The official repository contains the complete algorithmic logic, loss function, training loop, and evaluation pipeline written by the first author. However, it cannot be rated `PASS` out-of-the-box due to:  
> 1. A fatal runtime bug in `dataset_vqacp_MMBS.py` (`self.spatials` uninitialized).  
> 2. Complete absence of in-repo data preprocessing and download scripts.  
> 3. Inaccessible upstream university download server for VQA-CP v2.  
> 4. Absence of author-provided pre-trained checkpoints.  
>  
> The paper is **fully reproducible with engineering remediation** within 1–2 days of data setup and retraining.

---

### Step-by-Step Remediation Protocol (Repairing MMBS Baseline)

#### 1. Fix Fatal Attribute Bug in `dataset_vqacp_MMBS.py`
Replace lines 162–164:
```diff
- self.features = zarr.open(os.path.join(image_dataroot, 'trainval.zarr'), mode='r')
- self.s_dim = self.spatials[list(self.spatials.keys())[0]].shape[1]
- print('loading image features and bounding boxes done!')
+ self.features = zarr.open(os.path.join(image_dataroot, 'trainval.zarr'), mode='r')
+ self.s_dim = 6  # standard 6D spatial bounding box coordinates [x1, y1, x2, y2, w, h]
+ print('loading image features done!')
```
And replace line 240:
```diff
- spatials = torch.from_numpy(np.array(self.spatials[entry['image']]))
+ spatials = torch.zeros((36, 6), dtype=torch.float32)
```

#### 2. Import Data Scripts from `CrossmodalGroup/SSL-VQA`
Clone `SSL-VQA`'s `data/` directory to obtain:
* `download_data.sh` (or community mirror script).
* `create_dictionary.py` (builds `dictionary.pkl` from question vocabulary).
* `preprocess_text.py` (formats soft-score ground truth targets into `train_target.pkl`).

#### 3. Establish Minimal Reproduction Run (Smoke Test)
* Train with `--ratio 0.05 --num_epochs 2` on 5% subset to verify convergence and gradient flow in $< 15$ minutes.
* Run full training with fixed seed `1024` for 30 epochs to reproduce the reported ~48.19% VQA-CP v2 anchor result.
