---
name: generate-adjudication-examples
description: 'Generate corrected adjudication example files from de-identified annotation cases and meeting-minutes decisions. Use when: creating draft examples, correcting annotation labels, aligning examples with meeting rules, writing round examples, consolidating adjudication cases, building training examples for annotators.'
argument-hint: 'Provide de-identified example cases (pasted or attached) and the relevant meeting date or decisions'
---

# Generate Adjudication Examples from Cases + Meeting Decisions

## When to Use
- De-identified annotation examples with preliminary or conflicting labels need to be consolidated into a corrected example file.
- Meeting minutes contain annotation decisions that should override or refine the preliminary labels.
- Creating training/reference examples for annotators aligned to the latest agreed rules.

## Inputs
The user will provide:
1. **De-identified example cases** — pasted text or attached file containing annotation snippets with annotator-level labels (typically including columns: Annotator, Annotation Text, Span, Class, Assertion, Experiencer, Temporality, Severity/Burden/Certainty).
2. **Meeting decisions** — either:
   - A reference to a meeting minutes file (e.g., `meeting_minutes/20260528.md`), or
   - A meeting date so the skill can locate the relevant file, or
   - Decisions pasted directly.

## Output
A Markdown file saved to `draft_deid_examples/roundN_example.md` (round number inferred from context or asked from user).

## Procedure

### 1. Load the meeting decisions
- Read the referenced meeting minutes file from `meeting_minutes/`.
- Extract all numbered decisions from the **Key Decisions and Clarifications** section.
- Build a checklist of applicable rules (each rule = a label-correction trigger).

### 2. Parse the input examples
For each example case, extract:
- File identifier and date
- The clinical text snippet
- Each annotator's label row (Class, Assertion, Experiencer, Temporality, Severity, Burden, Certainty, etc.)
- The original explanation (if provided)

### 3. Apply meeting rules to correct labels
For each example, walk through the decision checklist and determine whether any rule changes the adjudicated label. Common correction patterns:

| Meeting Rule | Label Impact |
|---|---|
| Symptom in medication/template list only → ignore | Remove annotation or mark "No annotation" |
| Chronic non-resolving diagnosis → current | Change Temporality from Historical to Current |
| MMRC-like descriptors → infer burden | Set Burden to appropriate level (High/Moderate/Low) |
| Educational handout text → ignore | Remove annotation or mark "No annotation" |
| FEV1 excluded from severity classification | Set Severity to Unspecified |
| Encounters within 2 weeks → one episode | Merge exacerbation counts, note episode linkage |

If no rule applies, keep the majority/adjudicated label unchanged.

### 4. Write each example using the annotation-guideline format

Use the same bulleted-list style found in the project's XXXXXX_Annotation_Guidelines.md appendix. **Do not use tables.**

```markdown
N. "**annotated span or key phrase**" — surrounding context in plain text
    - Evidence Type: [Class]
    - Assertion: [Affirmed/Negated/…]
    - Temporality: [Current/Historical]
    - Experiencer: [Patient/Other]
    - [Severity/Certainty/etc. as applicable]: [value]

    [1-3 sentence explanation: which meeting rule applies, why labels were set this way, how annotator disagreement was resolved.]
```

For cases where the decision is to exclude the annotation entirely:

```markdown
N. "**key phrase**" — surrounding context in plain text
    - Decision: No annotation

    [Explanation referencing the specific meeting decision.]
```

Attribute ordering follows the guideline convention:
`Evidence Type → Assertion → Temporality → Experiencer → Severity → Certainty`

### 5. Write the file header
Summarize which meeting rules are in play for this round at the top of the file:

```markdown
# Round N Adjudication Examples (Corrected to YYYY-MM-DD Meeting Rules)

This file aligns labels/explanations to decisions documented in meeting minutes:
- [Rule 1 summary]
- [Rule 2 summary]
- ...
```

### 6. Save and confirm
- Save to `draft_deid_examples/roundN_example.md`.
- Report: number of examples processed, number of labels changed, which rules triggered corrections.

## Quality Checks
- Every corrected label must cite the specific meeting decision that justifies it.
- If the input example contains annotator disagreement, explain how it was resolved.
- Do not invent clinical context not present in the snippet or the meeting minutes.
- If an example does not match any meeting rule, keep the majority label and state "No rule-based correction; majority label retained."
- Preserve all de-identification — never attempt to reconstruct redacted identifiers.
- Use consistent attribute names matching the project schema (Class, Assertion, Experiencer, Temporality, Severity, Certainty).
