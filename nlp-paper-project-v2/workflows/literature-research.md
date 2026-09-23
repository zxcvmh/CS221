# Workflow: Literature Research

Goal: produce a traceable literature map without mixing source facts and interpretation.

Sequence:
`QUESTION → SEARCH → SOURCE VERIFY → EVIDENCE EXTRACT → CLUSTER → GAP ANALYZE → REVIEW`

Search priority:
1. ACL Anthology
2. official venue
3. official repository
4. arXiv
5. official dataset/model source
6. secondary sources for discovery/context only

Evidence record:
```yaml
evidence_id:
source_id:
source_type:
source_url:
location:
statement:
evidence_type: FACT | REPORTED_RESULT | IMPLEMENTATION_DETAIL
confidence:
notes:
```

Cluster by research problem, NLP mechanism, VQA failure mode, evaluation design, and limitations—not only model name.

For every proposed gap:
`OBSERVATION → EXISTING LIMITATION → MISSING CAPABILITY → TESTABLE NLP HYPOTHESIS`.

Send evidence + synthesis + claims to reviewer. Never send only an unstructured wall of prose.
