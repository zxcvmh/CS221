# Workflow: Reproducibility Audit

Goal: determine whether the selected paper can be reproduced sufficiently to justify extension.

Audit:
`PAPER → OFFICIAL CODE → ENVIRONMENT → DATA → CHECKPOINT → CONFIG → MINIMAL RUN → ANCHOR RESULT`

Artifact table:
| Artifact | Found? | Official? | Version | Blocking? | Evidence |
|---|---|---|---|---|---|
| Code | | | | | |
| Dataset | | | | | |
| Checkpoint | | | | | |
| Config | | | | | |
| Eval script | | | | | |
| Environment | | | | | |

Minimal run:
1. install environment;
2. tiny data subset;
3. one inference/training step;
4. verify output;
5. run evaluation;
6. record resource use.

Anchor result:
```yaml
paper_reported:
project_observed:
difference:
possible_causes:
```

Do not silently tune until the number matches.

Decision:
`PASS | PARTIAL | BLOCKED | REJECTED`.

Reviewer G2 inspects the audit and observed evidence.
