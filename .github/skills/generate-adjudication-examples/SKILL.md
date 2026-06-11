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
1. **De-identified example cases** — pasted text or attached file containing annotation snippets with annotator-level labels (typically including columns such as Annotator, Annotation Text, Span, Class/Label, and project-specific attributes).
2. **Meeting decisions** — either:
   - A reference to a meeting minutes file (e.g., `meeting_minutes/20260528.md`), or
   - A meeting date so the skill can locate the relevant file, or
   - Decisions pasted directly.

## Output
A Markdown file saved to `draft_deid_examples/roundN_example.md` (round number inferred from context or asked from user).

## Procedure

### 1. Discover the project's annotation schema
Before processing examples, identify the project's terminology:
1. Locate the canonical guideline file — root-level `.md` files excluding `README.md` and `CONTRIBUTING.md`.
2. Read the guideline to extract:
   - **Entity/class names** used in the schema (e.g., the labels annotators assign to spans).
   - **Attribute names and allowed values** beyond the common defaults (Assertion, Temporality, Experiencer). Look for project-specific attributes such as Severity, Burden, Certainty, or custom modifiers.
   - **Attribute ordering convention** used in the appendix examples.
3. Use these names and ordering consistently throughout the generated examples. When no appendix examples exist yet, default to: `Evidence Type → Assertion → Temporality → Experiencer → [project-specific attributes]`.

### 2. Load the meeting decisions
- Read the referenced meeting minutes file from `meeting_minutes/`.
- Extract all numbered decisions from the **Key Decisions and Clarifications** section.
- Build a checklist of applicable rules (each rule = a label-correction trigger).

### 3. Parse the input examples
For each example case, extract:
- File identifier and date (if present)
- The text snippet
- Each annotator's label row (class/label and all attributes present in the input)
- The original explanation (if provided)

### 4. Apply meeting rules to correct labels
For each example, walk through the decision checklist and determine whether any rule changes the adjudicated label. Common correction patterns include:

| Rule Pattern | Typical Label Impact |
|---|---|
| Mention appears only in boilerplate/template text → ignore | Remove annotation or mark "No annotation" |
| Context indicates a different temporal status than annotated | Change the temporal attribute to the correct value |
| A specific attribute value is excluded or redefined by a new rule | Update the attribute value accordingly |
| Multiple mentions should be merged into a single annotation | Consolidate and note linkage |
| Educational, instructional, or non-clinical text → ignore | Remove annotation or mark "No annotation" |

The specific rules depend entirely on the project and the meeting decisions. Use the decision checklist from Step 2 — do not assume any fixed set of correction patterns.

If no rule applies, keep the majority/adjudicated label unchanged.

### 5. Write each example using the annotation-guideline format

Match the formatting style found in the project's guideline appendix. If no appendix examples exist yet, use this default bulleted-list format. **Do not use tables for examples.**

```markdown
N. "**annotated span or key phrase**" — surrounding context in plain text
    - Evidence Type: [Class/Label]
    - Assertion: [Affirmed/Negated/…]
    - Temporality: [Current/Historical/…]
    - Experiencer: [Patient/Other/…]
    - [Additional project-specific attributes as applicable]: [Value]

    [1-3 sentence explanation: which meeting rule applies, why labels were set this way, how annotator disagreement was resolved.]
```

For cases where the decision is to exclude the annotation entirely:

```markdown
N. "**key phrase**" — surrounding context in plain text
    - Decision: No annotation

    [Explanation referencing the specific meeting decision.]
```

Use the attribute names and ordering discovered in Step 1.

### 6. Write the file header
Summarize which meeting rules are in play for this round at the top of the file:

```markdown
# Round N Adjudication Examples (Corrected to YYYY-MM-DD Meeting Rules)

This file aligns labels/explanations to decisions documented in meeting minutes:
- [Rule 1 summary]
- [Rule 2 summary]
- ...
```

### 7. Save and confirm
- Save to `draft_deid_examples/roundN_example.md`.
- Report: number of examples processed, number of labels changed, which rules triggered corrections.

## Quality Checks
- Every corrected label must cite the specific meeting decision that justifies it.
- If the input example contains annotator disagreement, explain how it was resolved.
- Do not invent context not present in the snippet or the meeting minutes.
- If an example does not match any meeting rule, keep the majority label and state "No rule-based correction; majority label retained."
- Preserve all de-identification — never attempt to reconstruct redacted identifiers.
- Use consistent attribute names matching the project's annotation schema (discovered in Step 1).
