# Workflow: Paper Deconstruction

Goal: understand a paper deeply enough to explain, reproduce, challenge, and extend it.

Pass 1 — Bibliographic/problem: publication identity, task, input/output, research question, assumptions.

Pass 2 — Mechanism:
`input → preprocessing → encoder → fusion/alignment → language reasoning → output`.
Record each component's function and evidence.

Pass 3 — Training/evaluation: datasets, splits, loss, optimizer, hyperparameters, baselines, metrics, ablations, results.

Pass 4 — Critical analysis:
- Why should it work?
- Which component is causally important?
- What does each ablation establish?
- What remains untested?
- Which failures matter to VQA?

Defense questions:
- What is the exact NLP bottleneck?
- Why is the intervention necessary?
- What changed between systems?
- Which result supports the mechanism?
- What would falsify the hypothesis?

Submit extraction and analysis separately to the reviewer.
