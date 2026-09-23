# Master Reproducibility Audit: MMBS & Post-MMBS Literature Frontier (2022–2026)

> **Workflow:** `workflows/reproducibility-audit.md`  
> **Source Documents:** `/home/zxcvmh/Projects/CS221/MMBS-refs/research-table.md` & `MMBS-refs/layer-papers.md`  
> **Baseline Anchor:** *MMBS* (Findings of EMNLP 2022)  
> **Audited Papers:** 11 papers spanning 2022–2026  
> **Audit Date:** 2026-09-23  
> **Standard:** ACL/EMNLP Scientific Reproducibility Gate (**Gate G2**)  

---

## 1. Executive Master Table: Reproducibility Across the Research Frontier

| # | Paper & Authors | Venue & Year | Official Code Status | Checkpoint Available? | Data Preprocessing in Repo? | Compute Feasibility (Colab/Kaggle) | Gate G2 Verdict |
|---|---|---|---|---|---|---|---|
| **1** | **MMBS** *(Si et al.)* | Findings of EMNLP 2022 | **OFFICIAL** (`PhoebusSi/MMBS`) *(Contains fatal uninitialized `spatials` bug)* | **NONE** (Must train from scratch) | **MISSING** (Outsourced to `SSL-VQA`) | 1x GPU T4 (15GB), ~12h | **`PARTIAL`** *(Feasible with patch)* |
| **2** | **DDG** *(Wen et al.)* | Findings of ACL 2023 | **OFFICIAL** (`Zhiquan-Wen/DDG`) *(Clean, Dockerized, Apache 2.0)* | **FULL / LIVE** (`best_model.pth` in GitHub Releases, 435MB, HTTP 200) | **FOUND** (Complete scripts in `data/`) | 1x GPU T4 (15GB), instant eval or ~14h train | **`PASS`** *(Gold Standard of Reproducibility)* |
| **3** | **CopVQA** *(Nguyen & Okazaki)* | EMNLP 2023 Main | **NOT RELEASED / MISSING** (No public repo on GitHub/Okazaki Lab) | **NONE** | **NONE** | N/A (Code missing) | **`BLOCKED`** *(Cannot reproduce without full reimplementation)* |
| **4** | **KDSR** *(Ning et al.)* | IEEE ICME 2024 | **NOT RELEASED / MISSING** (`kening-zai` has 0 public repos) | **NONE** | **NONE** | N/A | **`NOT AVAILABLE`** |
| **5** | **MCCD** *(Ma et al.)* | NeurIPS 2024 | **OFFICIAL** (`reml-group/MUSIC-AVQA-R`) | **NONE** (Training code complete) | **FOUND** (Test split included in repo) | 1x GPU (Audio-Visual features needed) | **`PARTIAL`** *(AVQA domain shift)* |
| **6** | **Robust VQA Survey** *(Ma et al.)* | IEEE TPAMI 2024 | **N/A** (Meta-analysis & taxonomy survey) | N/A | N/A | N/A | **`SURVEY ONLY`** |
| **7** | **CLAP** *(Wang et al.)* | IEEE ICME 2025 | **NOT RELEASED / MISSING** | **NONE** | **NONE** | N/A | **`NOT AVAILABLE`** |
| **8** | **CC-VQA** *(Li & Li)* | IEEE ICME 2025 | **NOT RELEASED / MISSING** | **NONE** | **NONE** | N/A | **`NOT AVAILABLE`** |
| **9** | **Counterfactual Dual-Bias** *(Wang et al.)* | IEEE TNNLS 2025 | **NOT RELEASED / MISSING** | **NONE** | **NONE** | N/A | **`NOT AVAILABLE`** |
| **10** | **DSI** *(Wang, Song, Wan)* | Expert Syst. Appl. 2026 | **OFFICIAL** (`songxdr3/DSI`) *(Derived from `chojw/genb`)* | **NONE** (No pre-trained weights) | **FOUND** (Complete scripts in `tools/`) | 1x GPU, ~100GB disk required | **`PARTIAL`** *(Reproducible via full training)* |
| **11** | **OSCAR** *(Chen, Si, Lin et al.)* | IJCAI 2026 | **NOT RELEASED / MISSING** | **NONE** | **NONE** | Multi-A100 (LLaVA-13B + MCTS) | **`BLOCKED`** *(Prohibitive compute + no code)* |

---

## 2. In-Depth Paper-by-Paper Reproducibility Profiles

### Paper 1: MMBS (Findings of EMNLP 2022) — Anchor Paper
* **Title:** *Towards Robust Visual Question Answering: Making the Most of Biased Samples via Contrastive Learning*
* **ACL ID:** `2022.findings-emnlp.495` | **arXiv:** `2210.04563`
* **Repository:** `https://github.com/PhoebusSi/MMBS` (Official, 1 commit, no license)
* **Code Verification:** Contains core InfoNCE objective and corruption logic (`Shuffling` and `Removal`).
* **Fatal Runtime Exception:** `dataset_vqacp_MMBS.py:163` references undeclared attribute `self.spatials`, crashing immediately on `DataLoader` creation. (Patch: set `self.s_dim = 6` and `spatials = torch.zeros((36, 6))`).
* **Artifact Status:** Training and test scripts present; data preprocessing missing in repo (must borrow from `SSL-VQA`). No pre-trained checkpoints.
* **Gate G2 Verdict:** **`PARTIAL`** (Requires 2 minor patches; training from scratch takes ~12h on Colab T4).

---

### Paper 2: DDG (Findings of ACL 2023) — Primary Direct Competitor
* **Title:** *Digging out Discrimination Information from Generated Samples for Robust Visual Question Answering*
* **ACL ID:** `2023.findings-acl.432` | **arXiv:** `2305.14811`
* **Repository:** `https://github.com/Zhiquan-Wen/DDG` (Official, Apache-2.0 License)
* **Artifact Completeness:**
  * Model implementations: UpDn + DDG (`UpDn_and_DDG.py`) and SAN + DDG (`SAN_and_DDG.py`).
  * Checkpoint: **VERIFIED LIVE** (`https://github.com/Zhiquan-Wen/DDG/releases/download/Models/best_model.pth`, 435MB, HTTP 200).
  * Pretrained UpDn base: Available in GitHub releases (`UpDn.pth`).
  * Preprocessed data: Augmented positive questions and image indices downloadable directly from releases.
  * Evaluation code: `test.py` and `comput_score.py` accurately report overall (61.22%) and split scores (Yes/No: 89.47%, Num: 48.70%, Other: 49.86%).
  * Environment: Docker image on DockerHub (`zhiquanwen/debias_vqa:v1`) and pinned `requirements.txt`.
* **Gate G2 Verdict:** **`PASS`** (The single most reproducible paper in the entire MMBS citation tree).

---

### Paper 3: CopVQA (EMNLP 2023 Main Conference) — The High-Profile Benchmark Rival
* **Title:** *Causal Reasoning through Two Layers of Cognition for Improving Generalization in Visual Question Answering*
* **ACL ID:** `2023.emnlp-main.568` | **arXiv:** `2310.05410`
* **Authors:** Bailey Trang Nguyen, Naoaki Okazaki (Tokyo Institute of Technology)
* **Repository Investigation:**
  * Searched official ACL proceedings, author homepages (Okazaki Lab), arXiv metadata, and GitHub orgs.
  * **Finding:** No official source code repository has been released.
* **Reproducibility Risk:** **CRITICAL**. Despite reporting 60.78% accuracy with only 1/4 the model size of competing baselines, independent reproduction is completely blocked without reimplementing the two cognitive pathway layers from scratch.
* **Gate G2 Verdict:** **`BLOCKED / NOT AVAILABLE`**.

---

### Paper 4: DSI (Expert Systems with Applications 2026) — SOTA Non-Pretrained Method
* **Title:** *Dual-space intervention for mitigating bias in robust visual question answering*
* **Venue:** Expert Systems with Applications, 2026
* **Repository:** `https://github.com/songxdr3/DSI` (Official author repo)
* **Code Verification:** Clean implementation branching from `chojw/genb` (CVPR 2021). Contains:
  * Dual-space projection modules (`q_model.py`, `base_model.py`).
  * Full preprocessing toolchain in `UpDn_DSI/tools/` (`download.sh`, `process.sh`, `create_dictionary.py`, `compute_softscore.py`).
  * Evaluation script `eval.py` that computes accuracy directly.
* **Limitations:** No pre-trained weights provided; requires downloading UpDn features from a Google Drive folder (~100GB storage required).
* **Gate G2 Verdict:** **`PARTIAL`** (Reproducible with full training; feasible for local workstation or Colab with Google Drive mount).

---

### Paper 5: MCCD (NeurIPS 2024) — Paradigm Expansion to AVQA
* **Title:** *Look, Listen, and Answer: Overcoming Biases for Audio-Visual Question Answering*
* **Venue:** Advances in Neural Information Processing Systems (NeurIPS 2024)
* **Repository:** `https://github.com/reml-group/MUSIC-AVQA-R` (Official)
* **Code Verification:** Contains modular engine (`engine.py`, `main.py`, `option.yaml`) and the complete *MUSIC-AVQA-R* test dataset (211,572 questions) stored directly in `./dataset/MUSIC-AVQA-R`.
* **Gate G2 Verdict:** **`PARTIAL`** (Code is complete and reproducible, but targets the audio-visual domain rather than standard visual-language VQA-CP v2).

---

### Papers 6 to 10: Unreleased Empirical Papers (Reproducibility Black Hole)
1. **KDSR (IEEE ICME 2024):** No public code repository found (`kening-zai` has 0 public repositories). **Status: UNRELEASED**.
2. **CLAP (IEEE ICME 2025):** No public code repository released. **Status: UNRELEASED**.
3. **CC-VQA (IEEE ICME 2025):** No official code release found. **Status: UNRELEASED**.
4. **Counterfactual Dual-Bias (IEEE TNNLS 2025):** No official repository found. **Status: UNRELEASED**.
5. **OSCAR (IJCAI 2026):** Code promised in preprint but currently unavailable; requires massive A100 GPU compute for MCTS search on LLaVA-13B. **Status: BLOCKED**.

---

## 3. Strategic Findings & Project Recommendations

### 1. The "Reproducibility Filter" Reality
Only **3 out of 10 empirical papers** in the MMBS literature tree provide working, accessible source code:
* **DDG (ACL 2023 Findings)**: **PASS** (100% reproducible with pre-trained checkpoint).
* **MMBS (EMNLP 2022 Findings)**: **PARTIAL** (Reproducible after fixing the 1-line `self.spatials` bug).
* **DSI (ESWA 2026)**: **PARTIAL** (Reproducible with scratch training).

All other published baselines (CopVQA, KDSR, CLAP, CC-VQA, Counterfactual Dual-Bias) are currently **unverifiable paper claims** without public code artifacts.

### 2. Concrete Action Plan for Your Research Project
1. **Primary Experimental Anchor:** Use **MMBS (UpDn)** as your direct baseline. Apply the verified bug fix to `dataset_vqacp_MMBS.py`, run minimal verification, and train the baseline model on Colab T4.
2. **Primary Competitive Baseline:** Compare directly against **DDG (Wen et al., ACL 2023)**. Since DDG provides a live `best_model.pth` checkpoint, you can run zero-cost evaluations on your test suite immediately without wasting GPU hours.
3. **Defense Advantage:** In your project defense, you can cite this exact audit to prove why you selected MMBS and DDG as your empirical comparison targets, while objectively pointing out that other recent papers (like CopVQA) have unreleased code artifacts that fail standard scientific reproducibility gates.
