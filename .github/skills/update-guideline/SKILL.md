---
name: update-guideline
description: 'Update an annotation guideline with new meeting decisions and matching examples. Use when: incorporating meeting minutes into guidelines, adding new annotation rules, appending adjudication examples to the appendix, syncing guidelines with recent decisions, updating a key-updates section.'
argument-hint: 'Provide the meeting minutes file (e.g., meeting_minutes/20260528.md) and optionally an adjudication examples file (e.g., draft_deid_examples/round6_example.md)'
---

# Update Annotation Guidelines from Meeting Decisions + Examples

## When to Use
- New meeting minutes contain annotation decisions that need to be incorporated into the project's canonical guideline Markdown file.
- Adjudication examples exist that illustrate the new decisions and should be added to the appendix.
- A meeting has concluded and the guidelines need to stay current with agreed rules.

## Identifying the Guideline File
This template repository uses a root-level Markdown file as the canonical guideline (e.g., `XX_Guideline(INCEpTION).md`). To locate it:
1. Look for root-level `.md` files **excluding** `README.md` and `CONTRIBUTING.md`.
2. If exactly one file remains, that is the guideline.
3. If multiple candidates exist, ask the user which file to update.

Throughout this skill, **"the guideline"** refers to whichever file is identified above.

## Inputs
The user will provide:
1. **Meeting minutes file** — a file in `meeting_minutes/YYYYMMDD.md` (or its content).
2. **Adjudication examples file** (optional) — a file in `draft_deid_examples/roundN_example.md` containing corrected examples aligned to the meeting decisions.

If only a meeting date is given, locate the corresponding files automatically.

## Output
Edits applied directly to the guideline file:
- A new entry in a **Key Updates** section (created if it does not exist).
- Selected examples appended to the appropriate subsections of **§ Appendix: Example Annotations**.

## Procedure

### 1. Discover the guideline structure
Before making any edits, read the guideline file and build a map of its sections. In particular, identify:
- **Concept definitions** — sections that define the annotation schema (entity types, attributes/modifiers, document-level labels). These may be titled "Task 1", "Explain concepts", etc.
- **Key Updates section** — a section for recording meeting-driven rule changes (may be titled "Key Updates from Recent Annotation Meetings", "Change Log", or similar). If no such section exists, one will be created in Step 3.
- **Appendix** — typically `## Appendix: Example Annotations`, with subsections organized by label/class. Note the existing subsection headings and the numbering of examples within each.
- **Project-specific terminology** — the entity class names, attribute names, and allowed values used by this project (e.g., `Evidence_type`, `Temporality`, `Assertion`, `Experiencer`, custom severity scales).

### 2. Load and parse the meeting decisions
- Read the meeting minutes from `meeting_minutes/YYYYMMDD.md`.
- Extract every numbered item from the **Key Decisions and Clarifications** section.
- For each decision, note the topic, the rule, and any example discussed.

### 3. Diff against the current guidelines
Check each meeting decision against the guideline's existing content:
- Any prior update entries (all `### YYYY-MM-DD Update` blocks or equivalent).
- The schema rules in the concept-definition sections (annotation rules, modifier definitions, inline examples).
- The appendix examples.

Classify each decision as one of:
| Status | Meaning | Action |
|---|---|---|
| **NEW** | Not mentioned anywhere in the current guidelines | Include in the update summary |
| **CLARIFICATION** | Partially covered but the meeting adds specificity (e.g., a concrete threshold, edge case, or exception) | Include in the update summary, noting it refines an existing rule |
| **ALREADY COVERED** | Fully captured by existing text | Skip — do not duplicate |

Only NEW and CLARIFICATION decisions go into the update entry.

### 4. Write the Key Updates entry

If the guideline already has a Key Updates section, insert a new `### YYYY-MM-DD Update` block at the **top** of that section (immediately after its `##` heading), so updates appear newest-first.

If no Key Updates section exists, create one:
- Insert `## Key Updates from Recent Annotation Meetings` immediately **before** the appendix section (or at the end of the file if there is no appendix).

Use this format:

```markdown
### YYYY-MM-DD Update (Meeting Type or Topic)
Summary: [1-2 sentence overview of what this update adds/changes.]

- **[Short rule title]**: [Clear, imperative statement of the rule. **Bold the key action phrase.**]
- **[Short rule title]**: [Next rule…]
```

Guidelines for writing each bullet:
- Write each rule as a standalone instruction an annotator can follow without reading the meeting minutes.
- Use the project's own annotation terminology exactly as it appears in the guideline (entity class names, attribute names, allowed values).
- If the rule references a threshold or boundary (time window, score cutoff, count), state the exact value.
- If the rule supersedes a prior decision, note which update it modifies.

### 5. Select examples from the adjudication file
If an adjudication examples file is provided (`draft_deid_examples/roundN_example.md`):

- For each NEW or CLARIFICATION decision from Step 3, find examples in the adjudication file that directly illustrate the rule.
- Prefer examples that:
  1. Demonstrate a rule that has **no existing example** in the appendix.
  2. Show a non-obvious edge case (e.g., "No annotation" decisions, boundary cases).
  3. Are distinct from examples already in the appendix (avoid near-duplicates).
- Skip examples that merely repeat a pattern already well-illustrated.

### 6. Determine insertion points in the appendix
The appendix is organized by label/class subsections. The exact subsection headings depend on the project's annotation schema — use the section map built in Step 1.

For each selected example:
1. Determine the correct subsection by matching the example's primary label (e.g., `Evidence Type`, `Class`, or `Decision`) to the corresponding appendix heading.
2. If no matching subsection exists and the label is new, create a new `### [Label] Examples` subsection in a logical position (alphabetical order or grouped with related labels).
3. For no-annotation cases, place them under a `### No-Annotation Examples` subsection (create it if it does not exist).

- Append each example at the **end** of its target subsection, continuing the existing numbering sequence.
- Preserve the formatting style already used in the appendix (bulleted lists, tables, or whatever convention the guideline follows).

### 7. Format each appended example
Match the style of existing appendix examples. If no examples exist yet, use this default format:

```markdown
N. "context text with **annotated span** highlighted"
    - Evidence Type: [Class]
    - Assertion: [Value]
    - Temporality: [Value]
    - Experiencer: [Value]
    - [Additional attributes as applicable]: [Value]

    [1-2 sentence explanation citing the meeting decision that motivates this example.]
```

For no-annotation cases:

```markdown
N. "context text with **key phrase** highlighted"
    - Decision: No annotation

    [Explanation referencing the specific meeting rule.]
```

Adapt attribute names and ordering to match the project's schema (use the terminology discovered in Step 1).

### 8. Apply edits and verify
- Use `multi_replace_string_in_file` to apply all insertions in a single operation when possible.
- After editing, re-read the modified sections to verify:
  - The new update block appears at the top of the Key Updates section.
  - Example numbering is sequential within each subsection.
  - No duplicate examples were introduced.
  - No existing content was accidentally removed.

### 9. Report summary
After completing the edits, report:
- Number of new decisions added to Key Updates.
- Number of examples appended, organized by subsection.
- Any decisions classified as ALREADY COVERED (and therefore skipped).
- Any adjudication examples that were skipped and why.

## Quality Checks
- Every bullet in the Key Updates entry must be traceable to a specific numbered decision in the meeting minutes.
- Do not add decisions that are already fully covered by existing guideline text.
- Every appended example must cite the meeting decision or rule it illustrates.
- Do not invent clinical context — use only text from the adjudication examples file.
- If the adjudication examples file is not provided, add only the Key Updates entry and note that examples can be added later when available.
- Preserve the existing formatting conventions of the guideline (bold spans, indentation style, blank lines between elements).
