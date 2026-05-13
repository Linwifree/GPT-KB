# KiroPrompt-GPT-Task-Packet-Template

用途：用于规范任务包的格式

格式：
**Markdown**，使用强 marker 包裹。

```text
<<<KIRO_PROMPT_TASK_PACKET_START>>>
# KiroPrompt-GPT Task Packet

...
<<<KIRO_PROMPT_TASK_PACKET_END>>>
```

## 必须包含的内容（本yaml仅为模板约束说明，具体格式参考下方的**建议 Markdown 骨架**）

```yaml
kiro_prompt_task_packet_requirements:
  format: "Markdown"
  wrapped_by:
    start: "<<<KIRO_PROMPT_TASK_PACKET_START>>>"
    end: "<<<KIRO_PROMPT_TASK_PACKET_END>>>"

  required_sections:
    - "Round"
    - "Task Title"
    - "Target Spec"
    - "Task Intent"
    - "Scope Boundary"
    - "Reading Scope"
    - "Expansion Permission"
    - "Expected Output"

  section_requirements:
    Round:
      must_include:
        - "Round ID"
        - "Creator: Leader-GPT"
        - "Recipient: KiroPrompt-GPT"

    Task_Title:
      purpose: "一句话说明本轮要生成什么 prompt-kiro.md。"
      must_not:
        - "不得写成“继续优化”“看一下相关内容”这类模糊标题。"

    Target_Spec:
      must_include:
        - "Spec Name"
        - "Spec Operation"
        - "Expected Spec Files"
      expected_spec_files:
        - "requirements.md"
        - "design.md"
        - "tasks.md"

    Task_Intent:
      purpose: "说明本轮希望 KiroPrompt-GPT 把什么 Leader-GPT 意图转成 prompt-kiro.md。"
      should_include:
        - "本轮目标"
        - "该 Spec 应解决什么问题"
        - "该 Spec 应推进到什么层级"
        - "该 Spec 与已有架构知识的关系"

    Scope_Boundary:
      must_include:
        - "In Scope"
        - "Out of Scope"
      purpose: "控制本轮 Spec 范围，避免大杂烩。"

    Reading_Scope:
      purpose: "指定 KiroPrompt-GPT 需要读取的文件范围。"
      must:
        - "每个文件必须使用 R? + D? 标注。"
        - "每个文件应带一句用途说明。"
      must_not:
        - "不得写“相关文件”“必要时参考”“看一下模块文稿”等模糊阅读要求。"

    Expansion_Permission:
      purpose: "说明本轮是否允许扩读。"
      must_include:
        - "是否允许扩读"
        - "允许扩读的触发条件"
        - "最多扩读次数"
      must_not:
        - "不得写“视情况扩读”。"

    Expected_Output:
      must_include:
        - "输出文件：prompt-kiro.md"
        - "输出应可直接发送给 Kiro IDE"
        - "输出必须聚焦目标 Spec"
      must_not:
        - "不得要求 KiroPrompt-GPT 输出 Review-GPT 审查内容。"
        - "不得要求 KiroPrompt-GPT 输出 round meta。"
        - "不得解释 KiroPrompt-GPT 应如何执行 R/D 阅读标准。"
```

## 建议 Markdown 骨架

```markdown
<<<KIRO_PROMPT_TASK_PACKET_START>>>
# KiroPrompt-GPT Task Packet

## 1. Round
- Round ID:
- Creator: Leader-GPT
- Recipient: KiroPrompt-GPT

## 2. Task Title
[一句话说明本轮任务]

## 3. Target Spec
- Spec Name:
- Spec Operation:
- Expected Spec Files:
  - requirements.md
  - design.md
  - tasks.md

## 4. Task Intent
[说明本轮希望 KiroPrompt-GPT 生成什么方向的 prompt-kiro.md]

## 5. Scope Boundary

### In Scope
- ...

### Out of Scope
- ...

## 6. Reading Scope
- `...`: R0 + D5（用途说明）
- `...`: R1 + D5（用途说明）
- `...`: R2 + D3（用途说明）

## 7. Expansion Permission
- Allow Expansion:
- Expansion Conditions:
  - ...
- Max Expansion Times:

## 8. Expected Output
- Output File: `prompt-kiro.md`
- Output Requirements:
  - 可直接发送给 Kiro IDE
  - 聚焦目标 Spec
  - 不生成无关说明
<<<KIRO_PROMPT_TASK_PACKET_END>>>
```