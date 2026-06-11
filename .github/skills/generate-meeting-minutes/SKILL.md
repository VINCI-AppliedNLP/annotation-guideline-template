---
name: generate-meeting-minutes
description: 'Generate structured meeting minutes from a raw meeting transcript. Focused on annotation-case decisions for the AGAVE Asthma/COPD NLP annotation project. Use when: converting transcript to minutes, summarizing annotation meeting, extracting annotation decisions from transcript, creating meeting notes.'
argument-hint: 'Paste or attach the raw meeting transcript text'
---

# Generate Meeting Minutes from Transcript

## When to Use
- A raw meeting transcript (pasted text, attached file, or transcript markdown) needs to be converted into structured meeting minutes.
- The meeting covers annotation adjudication, schema changes, labeling rules, or case-level decisions for the AGAVE Asthma/COPD NLP project.

## Input
The user will provide one of:
- Raw transcript text (pasted directly into chat)
- An attached transcript file
- A path to a transcript file in `meeting_transcripts/`

## Output
A Markdown file saved to `meeting_minutes/YYYYMMDD.md` (date extracted from transcript header or asked from user).

## Procedure

### 1. Parse the transcript
- Extract the meeting date from the transcript header or ask the user.
- Identify speakers and their roles (annotators, clinical expert, project lead).
- Flag segments where annotation cases, labeling rules, or schema decisions are discussed.

### 2. Draft the meeting minutes using this template

```markdown
# Meeting Minutes — AGAVE NLP Teamlet Meeting
**Date:** [Full date]  
**Duration:** ~[X] minutes

## Attendees
- [Names extracted from transcript]

## Agenda
- [Inferred from discussion topics]

## Progress Update
- [Any status updates on adjudication rounds, agreement rates, manuscript work]

## Key Decisions and Clarifications
[Number each decision. For each:]
1. [Short topic title]
   - Decision: [Clear, imperative statement of the rule agreed upon — bold the key phrase]
   - Rationale: [Why, if discussed]
   - Example discussed: [If a specific case was walked through, summarize it]

## Action Items
- [Person]
  - [Concrete task with enough context to act on]

## Timeline Notes
- [Any deadlines or availability constraints mentioned]

## Meeting Close
- [Summary of next steps]
```

### 3. Prioritize annotation-decision content
- The **Key Decisions and Clarifications** section is the most important output. Spend the most effort here.
- Each decision should be a standalone rule that annotators can apply without re-reading the transcript.
- Use the exact clinical/annotation terminology from the transcript (e.g., "Shared_Symptom", "COPD_Diagnosis", "Affirmed", "Historical", "MMRC").
- If a decision references a specific annotation case, include enough context (the text snippet, the disagreement, the resolution) so the rule is self-contained.

### 4. Handle common annotation decision categories
Watch for and structure decisions about:
- **Temporality** — When to label Current vs. Historical (chronic conditions, past medical history sections, resolved events)
- **Symptom evidence** — What counts as positive symptom evidence vs. template/medication-list noise
- **Diagnosis confirmation** — When a mention confirms diagnosis (radiology findings, educational text, impression sections)
- **Severity/burden** — How to assign severity or burden when explicit scores (FEV1, MMRC, CAT) are absent or excluded
- **Exacerbation counting** — Episode boundaries, hospitalization status, time-window rules
- **Schema changes** — New classes, attributes, or allowed values introduced

### 5. Cross-reference prior decisions
- Check existing files in `meeting_minutes/` for prior decisions on the same topic.
- If the new decision modifies or supersedes a prior one, note the change explicitly.

### 6. Save and confirm
- Save the file to `meeting_minutes/YYYYMMDD.md`.
- If a transcript file was provided, confirm whether the user also wants it saved/moved to `meeting_transcripts/`.
- Report a brief summary of the key decisions extracted.

## Quality Checks
- Every numbered decision must have a clear **Decision:** line with a bolded key phrase.
- Action items must name a responsible person.
- Do not fabricate decisions — if the transcript is ambiguous, flag it with `[NEEDS CLARIFICATION]` and note what was unclear.
- Do not include speaker-level dialogue; summarize into third-person statements.
