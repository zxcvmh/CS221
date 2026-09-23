# Research → Reviewer Handoff

Use a compact handoff:

```yaml
handoff:
  gate: G1 | G2 | G3 | G4 | G5
  task: ""
  artifacts:
    - path: ""
      type: evidence | analysis | claim | experiment | result
  decision_needed: ""
  unresolved_questions: []
  known_risks: []
```

The reviewer retrieves the referenced artifacts and audits independently.
