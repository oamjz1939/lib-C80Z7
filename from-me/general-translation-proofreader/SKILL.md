---
name: general-translation-proofreader
description: Proofread English translations against Chinese source text and an English reference section. Use when the user provides or asks to use fixed labeled sections [REFERENCE], [SOURCE_ZH], and [DRAFT_EN] for translation checking, terminology consistency, factual accuracy, and polished IELTS 7-level English output.
---

# General Translation Proofreader

Proofread the English draft by using the English reference as the terminology and style benchmark, and the Chinese source as the authority for meaning and facts.

## Required Input Format

Require the user input to contain all three uppercase labels:

```text
[REFERENCE]
English reference text, case, example, or model passage

[SOURCE_ZH]
Chinese source text

[DRAFT_EN]
English draft translation to proofread
```

If any label is missing, stop and state exactly which label is missing. Do not infer or proofread with incomplete sections.

Only `[DRAFT_EN]` is the text to revise and output. Do not rewrite `[REFERENCE]` or `[SOURCE_ZH]`.

## Workflow

1. Parse the three sections strictly by the labels `[REFERENCE]`, `[SOURCE_ZH]`, and `[DRAFT_EN]`.
2. Extract from `[REFERENCE]` any terminology, fixed expressions, proper nouns, document-specific phrasing, and field-specific style that should be reused.
3. Read `[SOURCE_ZH]` as the final authority for meaning, logic, facts, numbers, dates, names, amounts, and sequence.
4. Compare `[DRAFT_EN]` against `[SOURCE_ZH]` sentence by sentence.
5. Directly correct the English draft for:
   - mistranslation, omission, over-translation, or distorted logic;
   - terminology or phrasing that conflicts with `[REFERENCE]`;
   - changed numbers, dates, names, amounts, labels, or identifiers;
   - unnatural, overly casual, overly complex, or unclear English.
6. Keep the corrected English at approximately IELTS 7 level: precise, natural, academically appropriate when needed, but not unnecessarily difficult.
7. Preserve paragraph structure unless a small structural adjustment is required to make the translation faithful and readable.

## Output Format

Always output exactly two sections and no preface or closing note:

```markdown
**更改的内容**
- `original draft wording` -> `corrected wording` （简短原因）

**校对后的英文文段**
Corrected English passage
```

If there are no changes, output:

```markdown
**更改的内容**
无更改

**校对后的英文文段**
Original English draft
```

Write change reasons in Chinese. The corrected English passage must contain only the final English text.
