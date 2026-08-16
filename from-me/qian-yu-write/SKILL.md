---
name: qian-yu-write
description: 用于作业、个人陈述、申请文书、报告、邮件、简历和其他日常正式写作：通过协调器路由需求分析、结构设计、起草、批评与定稿子 Agent，保持真实材料、不编造事实，并在支持命名 subagent 的运行环境中隔离各角色提示词。
---

# Qian Yu Write

## Purpose

将日常写作任务拆成可控的多 Agent 工作流。主 Agent 负责识别任务、路由步骤和整合结果；具体角色负责分析、规划、写作、批评或编辑。

优先使用命名 subagent 或后端 Agent 注册表，让运行时根据 Agent 名称加载 `subagents/` 中对应的提示词。不要把这些提示词复制到协调器的 system prompt、Skill 内容或工具调用参数中。

## Workflow

先判断任务类型：

| 任务 | 调用顺序 |
|---|---|
| 从零开始写作 | `analyst` → `architect` → `writer` → `critic` → `editor` |
| 已有草稿需要修改 | `critic` → `editor` |
| 只需要结构或提纲 | `analyst` → `architect` |
| 只需要评价文本 | `critic` |
| 只需要语言润色 | `editor`，并明确只做语言层面的修改 |

默认按顺序执行，不让多个会修改同一文本的 Agent 并行运行。只有互不依赖的评价维度才可以并行，例如分别进行逻辑审查和语言审查。

## Coordinator Rules

- 维护一份共享任务状态：目标、读者、格式、长度、截止时间、限制、用户材料、原文和当前版本。
- 只向每个 Agent 传递完成当前步骤所需的数据，避免把无关历史或其他 Agent 的完整提示词传入上下文。
- 调用 Agent 时传递结构化任务对象，而不是让 Agent 自己猜测上下文。
- 不要自行扮演子 Agent，也不要把多个角色混在一次调用中。
- 不要要求 Agent 输出隐藏思考过程；要求简洁、结构化的结论、问题和修改建议。
- 缺少关键事实时标记为 `needs_input`，向用户提问；不要用猜测填空。
- 默认最多进行两轮“批评 → 编辑”循环；若没有高优先级问题或达到两轮，直接定稿。

## Shared Task Contract

使用以下字段组织 Agent 之间的数据，字段名可按宿主工具的 schema 映射：

```json
{
  "task_type": "new_writing | revision | critique | outline | polish",
  "goal": "用户想实现什么",
  "audience": "目标读者",
  "format": "文体、语言、长度和格式要求",
  "constraints": ["必须包含或禁止出现的内容"],
  "user_materials": ["用户提供的事实、经历、数据和观点"],
  "source_text": "已有文本，没有则为空",
  "previous_outputs": {}
}
```

下游 Agent 只能把 `user_materials` 视为事实来源。推测、建议或待确认信息必须明确标记，不能写成事实。

## Agent Boundaries

将 `subagents/analyst.md`、`subagents/architect.md`、`subagents/writer.md`、`subagents/critic.md` 和 `subagents/editor.md` 分别注册为命名 Agent。每个 Agent 只承担自己的职责：

- `analyst`：澄清目标、读者、限制、评价标准和信息缺口；不写正文。
- `architect`：设计中心论点、段落结构和材料安排；不写完整正文。
- `writer`：根据确认过的材料和结构生成草稿；不评价自己的草稿。
- `critic`：找逻辑、证据、受众、风格和可信度问题；不直接重写全文。
- `editor`：根据原文和审稿意见修改；不增加未经用户提供的事实。

## Subagent Injection Contract

按以下优先级绑定子 Agent 提示词：

### 1. 命名 Agent 或后端注册表

优先使用以下形式的后端接口：

```text
run_writing_agent(agent_name, task_object)
```

其中 `agent_name` 只能是 `analyst`、`architect`、`writer`、`critic` 或 `editor`；运行时根据名称绑定对应的 `subagents/*.md`，协调器只传递 `task_object`。

### 2. 路径绑定

如果宿主工具支持 `prompt_path`、`system_prompt_file` 或等价字段，可以只传递相对于本 Skill 根目录的提示词路径：

```text
run_writing_agent(
  agent_name="critic",
  prompt_path="subagents/critic.md",
  task_object={...}
)
```

运行时必须在子 Agent 的 system prompt 层读取该文件，再把 `task_object` 作为任务输入。优先使用相对路径，不要写死机器相关的绝对路径，例如 `C:\\...\\qian-yu-write\\subagents\\critic.md`。

路径绑定可以减少提示词进入协调器上下文的机会，但路径本身不是安全边界。只有当宿主工具明确把文件内容加载到子 Agent 的 system prompt 层时，才将其视为可靠的提示词注入。

如果宿主工具支持本地文件附件但不支持 `prompt_path`，将对应的 `subagents/*.md` 作为子 Agent 的 system-level 文件输入；不要只在普通任务文本中写“请读取这个文件”。

### 3. 降级模式

不要使用要求协调器自行拼接完整 system prompt 的接口，例如：

```text
spawn_agent(full_prompt, task)
```

如果宿主环境只有通用 `spawn_agent`，而没有命名 Agent 或后端注册表，先使用外部包装器、插件或 MCP 工具完成 prompt 绑定。若无法提供包装器，应明确这是降级模式：流程仍可执行，但无法保证子 Agent 提示词不出现在外层上下文中。

## Output Policy

定稿阶段只输出用户要求的最终文本，除非用户要求查看分析或修改说明。需要展示过程时，使用以下简洁结构：

```text
任务理解
结构方案
初稿
关键问题
最终版本
```

对作业、申请文书和个人陈述尤其要保留作者真实经历、语气和可验证细节；不要为了“降低 AI 味”而故意加入错误、口语化噪声或虚构的个人故事。
