---
name: memory-update-proposer
description: Propose and safely write user-confirmed project memory to project-memory.md. This skill is strictly user-initiated. It is triggered only when the user explicitly invokes it via a slash command (e.g. /memory) to propose a memory update, organize memory, or save preferences. The assistant must never proactively prompt or suggest updating the memory during ordinary conversation.
---

# Memory Update Proposer

Use this skill to turn durable project knowledge into user-confirmed entries in project-memory.md.

Core rule: do not write memory until the user has confirmed the exact final text that will be written. Do not show a diff preview.

## Trigger Policy

This skill is strictly user-initiated and is invoked when the user explicitly runs a slash command (such as /memory or /propose-memory).

The assistant must never proactively prompt, suggest, or offer memory updates to the user under any circumstances during normal conversation, even if the user states a recurring rule (e.g., "以后都这样"), changes a default configuration, or corrects assistant behavior.

All memory updates are handled exclusively in response to the user's explicit slash command trigger.

## What Belongs In Memory

Good memory entries are stable, actionable, and useful in future sessions:

- Project-wide conventions and constraints.
- User preferences that change future assistant behavior.
- Common commands, workflows, and environment assumptions.
- Repeated pitfalls or project-specific warnings.
- Decisions the user explicitly wants remembered.

Reject or avoid writing:

- Temporary task state, logs, transient errors, and one-time outputs.
- Unverified guesses or assumptions not accepted by the user.
- Large raw dumps, chat logs, or overly detailed implementation notes.

## Execution Workflow

1. Identify and Classify Candidates
Inspect the candidate durable memory and check project-memory.md (if it exists) to classify entries:
- New Entry (新增): A completely new rule.
- Update (更新): A modification or overriding of an existing rule. Do not simply append contradictory rules.
- Redundant (冗余): An exact duplicate of an existing rule. Quietly skip it and briefly notify the user.

2. Format and Show Proposal
Present the exact candidate entries using the [新增] or [更新] tags clearly. Do not show a diff preview.

3. Obtain Explicit Confirmation
Wait for the user's explicit confirmation using clear language such as "确认", "确认写入", "yes, write it". General positive remarks like "看起来不错" or "可以考虑" do not qualify as confirmation.

4. Execute Direct File Editing
After explicit confirmation, edit project-memory.md.
- Target only project-memory.md. Do not modify AGENTS.md or other instruction files.
- If project-memory.md does not exist, create it with "# Project Memory" followed by the confirmed entries.
- For additions, append them to the end of the file.
- For updates, locate the old target lines and replace them precisely.
- Preserve all other existing memory, do not introduce unconfirmed facts, and do not write temporary task states.

5. Report Result
Confirm the target file path and report the counts of entries written or updated.

## Confirmation Format

Use this format before writing. Dynamically adapt the language (Chinese/English) to match the user's communication language.

For pure additions:
```markdown
准备写入 project-memory.md：

- [新增] 本项目默认使用中文回复，除非用户明确要求其他语言。
- [新增] 记忆更新需要先整理为候选条目，经用户确认后再写入。

确认后我会把以上 2 条追加到 project-memory.md。
```

For updates or a mix of additions and updates:
```markdown
准备更新 project-memory.md：

- [更新] 将原规则 “测试默认使用 unittest 框架” 修改为 “测试默认使用 pytest 框架”。
- [新增] 追加项目默认的包管理器为 pnpm。

确认后我会执行以上修改。
```

If creating project-memory.md, say that confirmation will create the file.
