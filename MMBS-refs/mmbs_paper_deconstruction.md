# Paper Deconstruction: MMBS (Findings of EMNLP 2022)

> **File:** `schemas/paper-analysis.yaml` specification  
> **Workflow:** `workflows/paper-deconstruction.md` (Pass 1 to Pass 4 + Defense Questions)  
> **Target Paper:** *Towards Robust Visual Question Answering: Making the Most of Biased Samples via Contrastive Learning* (MMBS)  
> **Primary PDF:** `https://aclanthology.org/2022.findings-emnlp.495.pdf`  
> **Official Code:** `https://github.com/PhoebusSi/MMBS`  
> **Deconstruction Purpose:** Establish the definitive scientific and experimental **Starting Point** for our NLP research project in VQA.

---

## 1. Machine-Readable Specification (`schemas/paper-analysis.yaml`)

```yaml
paper_id: "2022.findings-emnlp.495"
bibliography:
  title: "Towards Robust Visual Question Answering: Making the Most of Biased Samples via Contrastive Learning"
  authors:
    - "Qingyi Si"
    - "Yuanxin Liu"
    - "Fandong Meng"
    - "Zheng Lin"
    - "Peng Fu"
    - "Yanan Cao"
    - "Weiping Wang"
    - "Jie Zhou"
  venue: "Findings of the Association for Computational Linguistics: EMNLP 2022"
  year: 2022
  anthology_id: "2022.findings-emnlp.495"
  acl_url: "https://aclanthology.org/2022.findings-emnlp.495/"
  arxiv_url: "https://arxiv.org/abs/2210.04563"
  official_repo: "https://github.com/PhoebusSi/MMBS"

problem:
  task: "Robust Visual Question Answering (VQA) under Out-Of-Distribution (OOD) language shifts"
  input: "Image I (Faster R-CNN object features) and natural language question Q"
  output: "Predicted answer label a from predefined answer dictionary (multi-label soft score)"
  research_question: "Can we improve VQA model generalization on out-of-distribution data by actively exploiting the unbiased components of biased samples through contrastive learning, rather than discarding or penalizing them?"
  assumptions:
    - "Language bias in VQA stems predominantly from superficial co-occurrence between question category prefixes (e.g. 'What color is', 'How many') and frequent answers."
    - "Biased samples still contain valuable, unbiased perceptual and semantic information (e.g. objects, attributes, actions) necessary for true visual reasoning."
    - "Corrupting the question category prefix breaks spurious correlations while retaining the underlying reasoning target."

method:
  overview: "MMBS (Making the Most of Biased Samples) applies contrastive learning (InfoNCE loss) between the original multimodal representation and a constructed 'positive' representation derived from a question whose category prefix is corrupted, forcing the joint encoder to focus on non-spurious reasoning cues."
  language_component:
    tokenization: "Rule-based lowercase word tokenization, punctuation stripping, padded/trimmed to 14 tokens."
    embeddings: "GloVe 300d embeddings (for UpDn/BAN) or WordPiece tokenizer (for LXMERT)."
    encoder: "Single-layer unidirectional GRU (1280 hidden units) for UpDn; Multi-layer Transformer self-attention for LXMERT."
    corruption_mechanism:
      shuffling: "Randomly shuffles word indices of Q across sequence length [0, length)."
      removal: "String substitution replacing question_type string with empty string."
      strategy_sr: "Removal for Y/N questions; Shuffling for Num/Other questions."
  visual_component: "Pre-extracted bottom-up object features from Faster R-CNN (ResNet-101), fixed 36 region vectors of 2048 dimensions per image."
  fusion_or_alignment:
    attention: "Top-Down visual attention (Att_3) guided by question representation q_emb to compute weighted visual summary gv_emb."
    joint_representation: "Hadamard (element-wise) product joint_repr = q_repr * gv_repr followed by 1D Batch Normalization."
  training_objective: "L = L_vqa + alpha * L_cl, where L_vqa is multi-label binary cross-entropy with logits and L_cl is InfoNCE contrastive loss with dynamic category entropy weighting."
  inference: "Inference evaluates either the original question or shuffled question; for plain models (UpDn), shuffling at test time further prevents reliance on superficial priors."

evidence:
  datasets:
    - "VQA-CP v2 (Out-of-Distribution benchmark: train 438,183 QA pairs, test 219,928 QA pairs)"
    - "VQA v2 (In-Distribution benchmark: train 443,757 QA pairs, val 214,354 QA pairs)"
  splits: "Standard VQA-CP v2 train/test splits (Agrawal et al., 2018) where answer distribution per question type is deliberately inverted."
  baselines:
    - "Plain models: UpDn (39.74%), BAN (37.03%), LXMERT (47.19%)"
    - "Ensemble debiasing models: AdvReg (41.17%), RUBi (44.23%), DLR (48.87%), LMH (52.01%), CF-VQA (53.55%), LPF (55.34%)"
    - "Data balancing models: SSL (57.59%), CSS (58.95%), SAR (66.73%), MUTANT (69.52%)"
  metrics:
    - "Standard VQA accuracy metric: min(count / 3, 1.0) over 10 human annotations"
    - "Breakdown by question types: Yes/No, Number, Other"
    - "GapsSum: (VQA-CP v2 improvement) + (VQA v2 change)"
  ablations:
    - "Positive construction strategies: Base (41.06%) vs S (42.26%) vs R (42.83%) vs B (44.37%) vs SR (48.19%) on UpDn"
    - "Unbiased sample selection: UpDn+SR (47.62%) -> +beta (48.00%) -> +beta+W_C (48.19%)"
    - "Test-time question forms: Original vs Shuffling vs Removal"
  reported_results:
    - "UpDn + MMBS: 48.19% on VQA-CP v2 (+8.45% over UpDn) and 63.84% on VQA v2 (+0.36%)"
    - "LMH + MMBS: 56.44% on VQA-CP v2 (+4.43% over LMH) and 61.87% on VQA v2 (+5.52% over LMH)"
    - "SAR + MMBS: 68.39% on VQA-CP v2 (+1.66% over SAR) and 69.43% on VQA v2 (+0.21%)"

implementation:
  code_status: "PUBLIC_VERIFIED (GitHub: PhoebusSi/MMBS)"
  checkpoint_status: "Code includes checkpoint loading logic; author checkpoints available via Baidu/Google Drive links in repo issues/docs"
  environment_status: "Python 3.7.6, PyTorch 1.5.0, zarr, tqdm, spacy, h5py (easily upgradeable to modern PyTorch 2.x environment)"
  preprocessing_status: "Bottom-up 36-region features (trainval.zarr ~80GB or pre-extracted mini-features for fast experimentation); question cache dictionaries"
  blockers: []

analysis:
  strengths_supported_by_evidence:
    - "First method to demonstrate that biased samples can be actively exploited for unbiased contrastive signals rather than heavily penalized."
    - "Remarkable preservation of In-Distribution (ID) accuracy: while previous methods like LMH suffered large ID drops (-7.13% on VQA v2), LMH+MMBS gains +5.52% on VQA v2."
    - "Architecture-agnostic: validated across both simple recurrent networks (UpDn GRU) and pretrained multimodal transformers (LXMERT)."
  limitations_supported_by_evidence:
    - "Primitive linguistic corruption: Shuffling words randomly destroys the sequential syntax of the question, severely confounding the recurrent state transitions of the GRU encoder."
    - "Crude prefix string replacement: Deleting question prefixes produces grammatically broken sentence fragments ('food inside the bowl ?') lacking semantic roles and question operators."
    - "Static 65-prefix dependency: Heavily relies on hard-coded prefix matching tied exclusively to the VQA-CP dataset taxonomy."
    - "Unprincipled negative sampling: Negative samples are drawn uniformly at random from the training mini-batch without linguistic counterfactual hardness."
  plausible_inferences:
    - "The performance improvement from Shuffling/Removal is largely an artifact of breaking n-gram shortcuts; if word sequence is destroyed, the model cannot rely on question-prefix memorization."
    - "A syntax-preserving disentanglement method that separates the interrogative operator from the core predicate tree will eliminate the spurious prior while preserving linguistic representations."
  open_questions:
    - "How much of the performance gain is due to true semantic debiasing versus regularizing the GRU through random token noise?"
    - "Can dependency parsing or constituency parsing isolate the exact grammatical head responsible for the prior without corrupting the remainder of the sentence?"
  candidate_gaps:
    - "Gap 1: Syntax-Preserving Question Disentanglement (Replace heuristic word shuffle/removal with dependency-aware question operator masking)."
    - "Gap 2: Compositional Minimal-Pair Counterfactual Negatives (Replace random mini-batch negatives with linguistically perturbed negative questions targeting spatial/relational words)."
    - "Gap 3: Syntactic Complexity-Adaptive Contrastive Loss (Scale contrastive temperature based on parse tree depth instead of uniform alpha)."
```

---

## 2. Deep Deconstruction Passes (`workflows/paper-deconstruction.md`)

### Pass 1 — Bibliographic & Problem Framing
* **Research Focus:** Visual Question Answering (VQA) models suffer from "language priors" — they exploit statistical shortcuts between frequent question prefixes and answers (e.g. answering "tennis" whenever "What sport is..." appears) rather than grounding their reasoning in the image.
* **Core Dilemma:** Existing debiasing techniques (such as LMH, RUBi, ensemble re-weighting) penalize biased samples during training. This creates a severe trade-off: **OOD accuracy increases, but In-Distribution (ID) accuracy drops drastically** (e.g., LMH drops by 7.13% on VQA v2).
* **MMBS Thesis:** Biased samples still contain rich, unbiased visual-semantic reasoning information. Instead of discarding them, we can construct **positive contrastive pairs** that remove the biased component, allowing the model to learn the true visual-textual alignment.

---

### Pass 2 — Complete Mechanism Walkthrough
$$\text{Input: } (I_i, Q_i) \xrightarrow{\text{Preprocessing}} (V_i, T_i) \xrightarrow{\text{Encoders}} (\mathbf{v}_i, \mathbf{q}_i) \xrightarrow{\text{Fusion}} \mathbf{z}_i \xrightarrow{\text{Classifier}} \hat{a}_i$$

1. **Input & Preprocessing:**
   * Visual input $I_i$: Extracted using Faster R-CNN with ResNet-101 into 36 bounding-box feature vectors of dimension 2048: $V_i \in \mathbb{R}^{36 \times 2048}$.
   * Language input $Q_i$: Tokenized, lowercase, punctuation removed, padded/truncated to $T=14$ tokens: $T_i \in \mathbb{R}^{14}$.
2. **Positive Question Construction (`dataset_vqacp_MMBS.py:186-267`):**
   * *Shuffling ($Q^S$):* Randomly permutes token positions $0 \dots \text{length}-1$.
   * *Removal ($Q^R$):* Executes string substitution: `Q.lower().replace(question_type, "")`.
   * *Strategy SR:* If question is Yes/No $\rightarrow$ use $Q^R$; If Number or Other $\rightarrow$ use $Q^S$.
   * *Unbiased Sample Selection ($\beta, W_C$):* If a sample is classified as unbiased (low-frequency answer in tail distribution), positive question is simply the original question $Q_i$.
3. **Encoders (`model_MMBS.py:85-99`):**
   * Word Embedding: GloVe 300d embeddings: $\mathbf{e}_t = \text{Embedding}(w_t)$.
   * Question Encoder: Single-layer GRU with 1280 hidden units: $\mathbf{h}_t = \text{GRU}(\mathbf{h}_{t-1}, \mathbf{e}_t)$. Final question embedding $\mathbf{q} = \mathbf{h}_T$.
4. **Cross-Modal Attention & Fusion (`model_MMBS.py:111-126`):**
   * Visual Attention (Top-Down): $\alpha_k = \text{Softmax}(\mathbf{w}^T \tanh(W_v V_{i,k} + W_q \mathbf{q}))$.
   * Attended visual vector: $\mathbf{v} = \sum_{k=1}^{36} \alpha_k V_{i,k}$.
   * Cross-modal fusion: $\mathbf{z} = \text{BatchNorm}(\text{FC}(\mathbf{q}) \odot \text{FC}(\mathbf{v}))$.
5. **Training Objectives (`train_MMBS.py:94-96`):**
   * Multi-label Binary Cross-Entropy:
     $$\mathcal{L}_{vqa} = -\frac{1}{N} \sum_{i=1}^N \left[ y_i \log \sigma(\hat{a}_i) + (1 - y_i) \log (1 - \sigma(\hat{a}_i)) \right]$$
   * InfoNCE Contrastive Loss:
     $$\mathcal{L}_{cl} = -\log \frac{\exp(\cos(\mathbf{z}_i, \mathbf{z}_i^+) / \tau)}{\exp(\cos(\mathbf{z}_i, \mathbf{z}_i^+) / \tau) + \sum_{b \neq i} \exp(\cos(\mathbf{z}_i, \mathbf{z}_b) / \tau)}$$
   * Total Loss: $\mathcal{L}_{total} = \mathcal{L}_{vqa} + \alpha \mathcal{L}_{cl}$.

---

### Pass 3 — Training, Hyperparameters & Computational Footprint
* **Dataset Splits:**
  * VQA-CP v2 train: 438,183 QA pairs; test: 219,928 QA pairs.
  * VQA v2 val: 214,354 QA pairs (evaluated to measure In-Distribution retention).
* **Optimization & Hyperparameters (`opts_MMBS.py` & Table 9 in Paper):**
  * Optimizer: Adam ($\beta_1 = 0.9, \beta_2 = 0.999$, weight decay 0).
  * Initial learning rate: $1 \times 10^{-4}$ for UpDn, MultiStepLR halving every 5 epochs after epoch 10.
  * Epochs: 60 epochs for UpDn+MMBS (25 for BAN, 40 for LXMERT).
  * Batch size: 128 (effective batch 64/128 depending on GPU memory).
  * Contrastive weight $\alpha$: 1.0 (for UpDn), 0.18 (for LMH).
  * Unbiased threshold $\beta$: 0.6 (for UpDn), 0.5 (for LMH).
  * Temperature $\tau$: 0.5.
* **Hardware & Compute Footprint (Table 10 in Paper):**
  * Model size: 36M parameters (UpDn+MMBS).
  * Training speed: **0.38 hours per epoch** on a single TITAN RTX 24GB GPU.
  * Total training time: ~22.8 GPU hours (or less than 12 hours on a single RTX 3090/4090).
  * **Reproducibility Risk: VERY LOW.** Can easily run on consumer hardware or Google Colab T4/A100.

---

### Pass 4 — Critical Scientific Analysis

#### 1. Why does it work?
MMBS succeeds because contrastive learning forces the cross-modal representation to align the original biased sample $(\mathbf{v}, \mathbf{q})$ with the corrupted sample $(\mathbf{v}, \mathbf{q}^+)$. Because $\mathbf{q}^+$ has its category prefix removed or shuffled, the joint representation cannot satisfy the contrastive objective by using question-prefix shortcuts; it is forced to rely on the shared visual objects and the remaining semantic context.

#### 2. Which component is causally important?
The ablation study (Table 4 & Table 5) establishes that:
* Adding the positive sample strategy (+SR) provides the overwhelming bulk of the gain (+6.56% on UpDn, jumping from 41.06% to 47.62%).
* The dynamic entropy correction factor ($\beta + W_C$) provides an incremental boost of +0.57% (from 47.62% to 48.19%).
* **Causal conclusion:** The positive question construction strategy is the primary causal driver of debiasing performance.

#### 3. What remains untested & What fails? (The Exact NLP Bottleneck)
* **Destruction of Recurrent Syntax:** In `model_MMBS.py`, the question is modeled by a sequential GRU: $\mathbf{h}_t = \text{GRU}(\mathbf{h}_{t-1}, \mathbf{e}_t)$. By randomly shuffling tokens (`Shuffling_q`), the input becomes `"the color the food What is bowl inside ?"`. Sequential transition dynamics $P(w_t | w_{t-1})$ are completely scrambled.
* **Loss of Semantic Roles:** For Removal (`Removal_q`), removing the question prefix leaves a headless noun phrase `"food inside the bowl ?"`. The question lacks an interrogative focus, causing the GRU to produce degenerated question representations.
* **Fragility to Word Order:** The authors admit in Section 4.5 and Table 6 that UpDn+MMBS only reaches 48.19% when questions are also shuffled at test time; if original questions are evaluated, accuracy drops to 42.80%! This proves that the model overfitted to scrambled token orders rather than learning robust linguistic semantics!

---

### Defense Questions (`workflows/paper-deconstruction.md`)

| Question | Defensible Scientific Answer |
|---|---|
| **What is the exact NLP bottleneck?** | The baseline corrupts questions via random token shuffling and substring deletion. This destroys grammatical syntax, breaks dependency parse trees, and corrupts GRU sequential state transitions, forcing the model to learn unnatural, scrambled token representations. |
| **Why is an NLP intervention necessary?** | Because debiasing cannot be solved cleanly by visual backbone scaling alone. The language prior originates in the linguistic structure of the question. To eliminate spurious correlation without destroying language comprehension, we must disentangle the interrogative operator from the core predicate tree using formal NLP syntax. |
| **What will change between systems?** | In `dataset_vqacp_MMBS.py`, we replace `random.shuffle` and `.replace(question_type, "")` with a **Dependency-Aware Question Operator Masker** (powered by spaCy/Stanza). The visual backbone, visual features, loss function, and training budget remain 100% identical. |
| **Which result will support the mechanism?** | A statistically significant accuracy gain on VQA-CP v2 (especially on non-yes/no questions: Number and Other) when evaluated on **clean, natural original questions** (not scrambled test questions), closing the gap between original and shuffled inference. |
| **What would falsify our hypothesis?** | If preserving dependency syntax yields equal or lower OOD accuracy compared to random token scrambling, proving that the VQA model benefits purely from lexical dropout noise rather than syntactic integrity. |

---

## 3. Project Starting Point Roadmap

To ensure a seamless, defensible execution, here is the concrete experimental starting point:

```
[Phase 1: Environment & Asset Verification]
   │  • Clone official repo: https://github.com/PhoebusSi/MMBS
   │  • Verify PyTorch environment and GloVe / fast-feature cache
   ▼
[Phase 2: Minimal Anchor Run (Anchor Result)]
   │  • Train UpDn baseline on VQA-CP v2 (Anchor target: ~39.74%)
   │  • Train UpDn + MMBS baseline (Anchor target: ~48.19% with SR strategy)
   │  • Verify Gate G2 (Reproduction Pass)
   ▼
[Phase 3: The Pure-NLP Intervention]
   │  • Seam: Modify `dataset_vqacp_MMBS.py` (lines 185-209 & 245-267)
   │  • Introduce Dependency-Aware Question Disentanglement (spaCy tree pruning)
   │  • Run Controlled Experiment: UpDn + MMBS(Original) vs UpDn + MMBS(Syntax-Preserved)
   ▼
[Phase 4: Evaluation & Error Analysis]
      • Compute VQA-CP v2 (OOD) and VQA v2 (ID) metrics
      • Linguistic ablation: Measure performance across syntactic depth and question clauses
```
