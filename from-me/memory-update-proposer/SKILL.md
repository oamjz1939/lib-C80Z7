---
name: memory-update-proposer
description: Propose and safely write user-confirmed project memory to project-memory.md. Use when the user explicitly asks to整理记忆, update project memory, save a durable project rule, write a preference to memory, or generate a memory update proposal. Invoke proactively only for rare, high-value durable rules such as "以后都这样", "this project defaults to...", or "do not do this again"; do not use for ordinary task status, temporary facts, secrets, or unverified guesses.
---

# Memory Update Proposer

Use this skill to turn durable project knowledge into user-confirmed entries in `project-memory.md`.

Core rule: do not write memory until the user has confirmed the exact final text that will be written. Do not show a diff preview.

## Trigger Policy

Default to user-initiated use. Treat these requests as explicit triggers:

- "整理记忆"
- "更新项目记忆"
- "把这条写进记忆"
- "生成记忆提案"
- "保存这个偏好"
- "write this to project memory"

Proactively offer this skill only when the information is clearly durable and likely to affect future collaboration, such as:

- The user states a future rule: "以后都这样", "always do this", "never do that".
- The user defines a project default: language, workflow, test command, architecture constraint, or tool preference.
- The user corrects recurring assistant behavior in a way that should persist.

Do not proactively offer memory updates for ordinary progress, one-off command output, temporary debugging state, speculative conclusions, implementation details unlikely to recur, or any sensitive material.

## What Belongs In Memory

Good memory entries are stable, actionable, and useful in future sessions:

- Project-wide conventions and constraints.
- User preferences that change future assistant behavior.
- Common commands, workflows, and environment assumptions.
- Repeated pitfalls or project-specific warnings.
- Decisions the user explicitly wants remembered.

Reject or avoid writing:

- Passwords, tokens, credentials, private keys, or personal sensitive data.
- Temporary task state, logs, transient errors, and one-time outputs.
- Unverified guesses or assumptions not accepted by the user.
- Large raw dumps, chat logs, or overly detailed implementation notes.

## Required Workflow

1. Identify candidate durable memory.
2. Check existing `project-memory.md` (if it exists) to inspect for duplicate, overlapping, or conflicting rules.
3. Convert the candidate memory into concise Markdown bullet entries, identifying whether each is a:
   - **New Entry (新增)**: A completely new rule.
   - **Update/Replacement (更新)**: A modification or overriding of an existing rule.
   - **Redundant (冗余)**: An exact duplicate of an existing rule (these should be quietly skipped/discarded with a brief note to the user).
4. Show the exact final text and action type (e.g., [新增], [更新]) that will be written.
5. Ask for explicit write confirmation.
6. Write only after confirmation.
7. Report the file path and number of entries written or updated.

The user must confirm with clear language such as "确认", "确认写入", "可以写入", "yes, write it", or equivalent. Phrases such as "看起来不错", "可以考虑", or "maybe" are not write confirmation.

## Confirmation Format

Use this format before writing. Dynamically adapt the language (Chinese/English) to match the user's communication language.

For pure additions:
```markdown
准备写入 `project-memory.md`：

- [新增] 本项目默认使用中文回复，除非用户明确要求其他语言。
- [新增] 记忆更新需要先整理为候选条目，经用户确认后再写入。

确认后我会把以上 2 条追加到 `project-memory.md`。
```

For updates or a mix of additions and updates:
```markdown
准备更新 `project-memory.md`：

- [更新] 将原规则 “测试默认使用 unittest 框架” 修改为 “测试默认使用 pytest 框架”。
- [新增] 追加项目默认的包管理器为 pnpm。

确认后我会执行以上修改。
```

If creating `project-memory.md`, say that confirmation will create the file.

## Writing Rules

Use `project-memory.md` as the fixed target. Do not default to modifying `AGENTS.md` or other instruction files.

Default operations:
- **Redundant Check**: If the candidate rule is identical in meaning or text to an existing entry in `project-memory.md`, explain to the user that it is already recorded and skip writing it.
- **Conflict & Overwrite Check**: If a new candidate rule contradicts or updates an existing entry (e.g. tech stack, architecture, workflow, or behavior preferences), treat it as an `[更新]` operation. Do not simply append it, as that creates duplicate contradictory entries.
- **New Entry Check**: If the candidate rule is completely new, treat it as an `[新增]` operation.

Allowed operations:

- Create `project-memory.md` when it does not exist and the user confirms the final initial content.
- Append confirmed `[新增]` entries to the end of the file.
- Perform targeted edits or partial rewrites to replace/remove specific existing entries for confirmed `[更新]` entries.
- Rewrite the whole file only when the user explicitly asks to reorganize, merge, delete, or rewrite memory and confirms the full replacement text.

When rewriting or editing:

- Preserve existing memory unless the user explicitly requested removal or replacement.
- Do not introduce new facts that were not confirmed.
- Do not convert temporary task state into long-term memory.
- Present the complete replacement text before asking for confirmation.

## Direct File Editing

Use the host environment's normal file editing tools to update `project-memory.md` after explicit confirmation. Do not use a helper program. Do not show a diff preview unless the user explicitly asks for one.

Default behavior:

- If `project-memory.md` does not exist, create it with `# Project Memory` followed by the confirmed entries.
- If `project-memory.md` exists:
  - For `[新增]` entries, append them to the end of the file.
  - For `[更新]` entries, locate the old target lines in `project-memory.md` and replace them precisely with the new content, or perform a clean edit of the file.
- If the user explicitly requests deletion, merging, reorganization, or full rewrite, show the complete replacement text first and wait for explicit confirmation before editing.

When editing directly:

- Write only the exact entries or complete replacement text the user confirmed.
- Do not add unconfirmed facts while editing.
- Do not remove existing memory unless the user explicitly requested removal or approved an replacement update.
- Do not edit `AGENTS.md` or other instruction files unless the user separately requests that outside this skill's memory-writing workflow.
