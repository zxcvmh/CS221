# Workflow: Experiment Design

Goal: answer an NLP research question inside VQA.

Sequence:
`FAILURE MODE → HYPOTHESIS → INTERVENTION → CONTROLS → METRIC → EXPERIMENT → ERROR ANALYSIS`

Research-question template:

> Under [VQA condition], does [NLP intervention] improve [language behavior/task metric] while holding [controls] constant?

Experiment card:
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
uncertainty:
acceptance_criteria:
```

Minimum experiment set:
1. baseline;
2. intervention;
3. ablation/control;
4. stratified evaluation;
5. qualitative error analysis.

If A and B both change, the result cannot safely be attributed to A.

For VQA, stratify by the targeted failure mode where justified (e.g. OCR-heavy, numerical, compositional, spatial-language).

Reviewer G4 must inspect causal interpretability before expensive training.
