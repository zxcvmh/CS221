---
name: nlp-paper-project
description: ACL-centered research skill for a VQA application project in which NLP is the primary scientific contribution. Covers literature discovery, source verification, paper extraction, reproducibility auditing, controlled experimentation, VQA evaluation, error analysis, and adversarial review.
---

# NLP Paper Project Skill

## 0. Mission

Build a defensible NLP research project whose application domain is VQA.

`VQA application → language-centric bottleneck → NLP research question → ACL-family method/paper → reproducibility audit → controlled reproduction → NLP intervention → VQA evaluation → error analysis → evidence-backed conclusion`

Optimize for scientific validity and reproducibility, not the largest model or highest-looking score.

## 1. Non-negotiable constraints

### 1.1 Main-paper provenance
The main paper must be verifiable on ACL Anthology.

Required:
- exact title, authors, venue/year;
- Anthology ID;
- direct ACL Anthology page;
- arXiv link if available;
- official code repository if available;
- provenance of checkpoint/data/configuration when relevant.

Valid ACL-family venues may include ACL, EMNLP, NAACL, EACL, AACL, CoNLL, TACL, and Computational Linguistics, subject to the course requirement.

### 1.2 Reproducibility gate

```text
ARTIFACT AUDIT → ENVIRONMENT AUDIT → DATA/CHECKPOINT AUDIT → MINIMAL RUN → ANCHOR RESULT → DECISION
```

Allowed: `PASS`, `PARTIAL`, `BLOCKED`, `REJECTED`.

`PARTIAL` is not permission to claim full reproduction.

### 1.3 Contribution boundary

Not sufficient alone:
- backbone replacement;
- larger model;
- quantization;
- prompt wording;
- UI/API;
- ordinary engineering optimization;
- library combination.

A defensible contribution connects:

`observed VQA/NLP failure → research gap → NLP hypothesis → controlled intervention → measurable effect`.

## 2. Source hierarchy

1. ACL Anthology
2. official conference/journal
3. official author/project repository
4. official dataset/model repository
5. arXiv
6. Hugging Face/package registry
7. secondary sources

When sources conflict, record the conflict and identify the authoritative source. Do not silently reconcile.

## 3. Literature discovery

For every candidate create:

```yaml
paper_id:
title:
authors:
venue:
year:
anthology_id:
acl_url:
arxiv_url:
official_repo:
task:
dataset:
method:
language_component:
vqa_relevance:
code_status:
checkpoint_status:
reproducibility_risk:
evidence_ids:
status:
```

Do not select from abstract relevance alone. Screen provenance and reproducibility first.

## 4. Paper extraction

Extract evidence before analysis.

### A. Bibliographic
Title, authors, venue/year, Anthology ID, URLs.

### B. Problem
Task, input/output, bottleneck, research question, assumptions.

### C. Method
Architecture, language component, visual component, training objective, inference, important hyperparameters.

### D. Evidence
Datasets, splits, baselines, metrics, ablations, reported results, limitations.

### E. Implementation
Official repo, commit/version, environment, checkpoints, preprocessing, missing artifacts, blockers.

Never mix extraction and interpretation in the same field.

## 5. Analysis

Ask:
1. What does the paper claim?
2. Why should the mechanism cause the effect?
3. What evidence supports it?
4. What alternative explanations exist?
5. What does the paper not establish?

Label statements: `FACT | INFERENCE | ASSUMPTION | HYPOTHESIS | RECOMMENDATION | UNKNOWN`.

Always distinguish:
`paper claim ≠ reproduced result ≠ project result ≠ interpretation`.

## 6. VQA × NLP framing

Start from a language-centric VQA failure mode:
- OCR/text grounding;
- entity ambiguity;
- multilingual/multimodal alignment;
- question understanding;
- compositional reasoning;
- spatial language;
- temporal language;
- numerical/text reasoning;
- answer normalization;
- context selection;
- hallucinated textual evidence.

Preferred question form:

> Under failure condition Y, does NLP intervention X improve behavior Z while holding visual backbone and evaluation conditions constant?

## 7. Controlled experiment contract

Every experiment records:

```yaml
experiment_id:
research_question:
hypothesis:
independent_variable:
dependent_variable:
controls:
dataset:
split:
model:
checkpoint:
prompt:
preprocessing:
seed:
hardware:
software_commit:
metric:
result:
uncertainty:
limitations:
decision:
```

Control dataset/split, visual backbone, language backbone, checkpoint, decoding, prompt, preprocessing, evaluator, and seed policy as appropriate. Record every intentional change.

## 8. Evaluation

Use, when feasible:
- quantitative metrics;
- stratified evaluation by relevant failure type;
- qualitative error analysis.

A single aggregate score is insufficient when the claimed mechanism concerns language behavior.

## 9. Error analysis

```yaml
case_id:
question:
image_context:
gold_answer:
prediction:
failure_category:
linguistic_issue:
visual_issue:
alignment_issue:
reasoning_issue:
evidence:
counterfactual_fix:
```

Do not label a visual failure as an NLP failure without evidence.

## 10. Research gap

```text
OBSERVATION
→ FAILURE MODE
→ LIMITATION OF EXISTING METHOD
→ RESEARCH GAP
→ NLP HYPOTHESIS
→ CONTROLLED INTERVENTION
→ VQA EFFECT
```

A gap is not merely “the authors did not try model X.”

## 11. Claim protocol

```yaml
claim_id:
statement:
claim_type: FACT | INFERENCE | HYPOTHESIS | RECOMMENDATION
evidence_ids:
source_ids:
source_location:
inference_steps:
confidence:
counterevidence:
status:
```

No citation laundering and no presenting later project results as paper results.

## 12. Reviewer interface

Send structured artifacts:
1. evidence;
2. paper analysis;
3. claims;
4. experiment design/results;
5. current decision.

Reviewer audits `SOURCE → EXTRACTION → ANALYSIS → CLAIM → REPRODUCTION → EXPERIMENT → APPLICATION VALUE`.

## 13. Hardware discipline

Start with smoke tests. Measure VRAM. Use quantization/checkpointing only when justified and recorded. Record software versions and commits. Do not make an experiment fit at the expense of interpretability.

## 14. Language policy

User-facing: Vietnamese by default. Research artifacts/task specs/experiment records/claims/review reports: English by default unless another format is required.

## 15. Truthfulness

Distinguish:
`KNOWN | UNKNOWN | NOT FOUND | UNVERIFIED | CONFLICTING | OBSERVED | REPORTED | INFERRED`.

Never invent papers, repos, IDs, results, or reproduction status. Never tune toward a desired result. Never change evaluation conditions after seeing results without disclosure.

## 16. Completion standard

Complete only when the question, evidence, provenance, assumptions, reproducible artifacts, objective acceptance criteria, reviewer status, and next decision are explicit.
