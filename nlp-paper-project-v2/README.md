# NLP Paper Project — Research Operating System

## Mission
This project supports a **VQA application project with NLP as the primary scientific contribution**.

Research chain:
`VQA application → language-centric bottleneck → NLP research question → ACL-family paper/method → reproducibility audit → controlled reproduction → NLP intervention → VQA evaluation → error analysis → evidence-backed conclusion`

**Evidence before conclusion. Reproducibility before extension. NLP contribution before engineering novelty.**

## 1. What to load at chat start

Do not paste every project file into every chat.

Progressive disclosure:

```text
ALWAYS
README.md
SKILL.md

THEN — only the current workflow
workflows/topic-discovery.md
workflows/literature-research.md
workflows/paper-deconstruction.md
workflows/reproducibility-audit.md
workflows/experiment-design.md
workflows/final-defense.md

WHEN REVIEW IS REQUIRED
agents/nlp-paper-project-reviewer.md

WHEN STRUCTURED EVIDENCE IS PRODUCED
schemas/evidence.yaml
schemas/claim.yaml
schemas/paper-analysis.yaml
schemas/review.yaml
```

README = orientation. SKILL = operating policy. Workflow = procedure. Reviewer = independent auditor. Schemas = machine-readable contracts.

## 2. Chat bootstrap

If the framework automatically loads `SKILL.md`, do not paste it again. Give a compact instruction:

```text
Load and follow:
- README.md
- SKILL.md
- workflows/<current-workflow>.md

Current project state:
<one paragraph or project-state path>

Current task:
<exact research task>

Constraints:
- Main paper must be verified on ACL Anthology.
- Reproduction must pass the reproduction gate before extension.
- VQA is the application domain; NLP is the main scientific contribution.
- Do not treat backbone replacement alone as research novelty.
- UNKNOWN and NOT FOUND are valid outcomes.

Return structured artifacts, not unsupported prose.
```

## 3. Two-agent architecture

Keep the system intentionally small.

### Agent 1 — `nlp-paper-project`
Researcher / Builder:
- discover and verify papers;
- audit artifacts;
- extract evidence;
- deconstruct methods;
- formulate research questions;
- design/reproduce experiments;
- analyze results;
- produce structured evidence and claims.

### Agent 2 — `nlp-paper-project-reviewer`
Independent Scientific Auditor:
1. Source Audit
2. Extraction Audit
3. Analysis Audit
4. Claim Audit
5. Reproducibility Audit
6. Experiment Audit
7. VQA/NLP contribution audit

No third reviewer is required.

## 4. Decision gates

| Gate | Question | Output |
|---|---|---|
| G1 | Is the paper a valid candidate? | APPROVE / BLOCK / REJECT |
| G2 | Is reproduction sufficient? | PASS / PARTIAL / BLOCKED / REJECTED |
| G3 | Is there a real NLP research question? | APPROVE / REQUEST_CHANGES / REJECT |
| G4 | Is the experiment fair and interpretable? | APPROVE / REQUEST_CHANGES / BLOCK |
| G5 | Are results and claims supported? | APPROVE / REQUEST_CHANGES / BLOCK |

Never skip G2 because a paper looks interesting.

## 5. Evidence discipline

```text
CLAIM → EVIDENCE → SOURCE → SOURCE LOCATION → INFERENCE (if any) → CONFIDENCE
```

Use: `FACT`, `INFERENCE`, `ASSUMPTION`, `HYPOTHESIS`, `RECOMMENDATION`, `UNKNOWN`, `NOT_FOUND`.

`NOT FOUND` does not mean `DOES NOT EXIST`.

## 6. Definition of done

A phase is complete only when:
- the research question is explicit;
- evidence is recorded;
- provenance is checked;
- assumptions are visible;
- artifacts exist;
- acceptance criteria are objective;
- reviewer status is recorded;
- the next decision is explicit.

A polished paragraph is not evidence of completion.
