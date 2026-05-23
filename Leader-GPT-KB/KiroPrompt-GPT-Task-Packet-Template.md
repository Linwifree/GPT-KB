
# KiroPrompt-GPT Task Packet Template

用途：
本文件定义 Leader-GPT 生成 `commitLeaderArtifacts.task_packet_md` 时必须遵守的 Markdown 结构、字段语义和边界规则。

`task_packet_md` 是 Leader-GPT 提交给 Gateway 的三个产物之一。  
它不是聊天输出，不是独立文件写入口，而是 `commitLeaderArtifacts` requestBody 中的一个 Markdown string 字段。  
Gateway 接收后会将其保存为状态层中的 `task-packet.md`，供后续 KiroPrompt-GPT 使用。

---

## 1. Artifact Identity

```yaml
system_facts:
  file_identity:
    id: "kiro-prompt-gpt-task-packet-template"
    name: "KiroPrompt-GPT Task Packet Template"
    owner_role: "Leader-GPT"
    use_stage: "before_commitLeaderArtifacts"
    purpose: "定义 commitLeaderArtifacts.task_packet_md 的 Markdown 正文结构。"

  artifact_identity:
    action_field: "task_packet_md"
    format: "Markdown string"
    submitted_via: "commitLeaderArtifacts"
    stored_by_gateway_as: "task-packet.md"
    future_consumer: "KiroPrompt-GPT"

  relation_to_gateway:
    - "Leader-GPT 每轮先调用 getLeaderContext。"
    - "Leader-GPT 根据 mode、input_type、expected_action 和 context 生成 task_packet_md。"
    - "Leader-GPT 通过 commitLeaderArtifacts 一次性提交 task_packet_md、review_intent_pack_md、leader_work_summary_json。"
    - "task_packet_md 必须与 leader_work_summary_json.file_change.target_file 保持一致。"

  scope_boundary:
    owns:
      - "本轮给 KiroPrompt-GPT 的任务目标。"
      - "唯一 target_file。"
      - "file_context。"
      - "Leader-GPT 的任务意图。"
      - "Scope Boundary。"
      - "Reading Scope。"
      - "Expansion Permission。"
      - "Expected Output。"

    does_not_own:
      - "不定义 Review-GPT 的完整审查制度。"
      - "不定义 leader_work_summary_json 的 schema。"
      - "不生成 round_id。"
      - "不生成 leader-output-meta.json。"
      - "不写 Gateway 内部状态。"
      - "不决定 routing、next_actor、lock 或 status。"
      - "不要求 KiroPrompt-GPT 修改状态层。"
````

---

## 2. Required Wrapper

`task_packet_md` 必须使用强 marker 包裹：

```text
<<<KIRO_PROMPT_TASK_PACKET_START>>>
# KiroPrompt-GPT Task Packet

...
<<<KIRO_PROMPT_TASK_PACKET_END>>>
```

```yaml
format_rules:
  format: "Markdown"
  wrapped_by:
    start: "<<<KIRO_PROMPT_TASK_PACKET_START>>>"
    end: "<<<KIRO_PROMPT_TASK_PACKET_END>>>"

  must:
    - "正文必须是非空 Markdown string。"
    - "必须包含唯一 Target File。"
    - "Target File 必须与 leader_work_summary_json.file_change.target_file 完全一致。"
    - "Target File 必须与 review_intent_pack_md 中的 Review Target 完全一致。"
    - "所有 Reading Scope 文件必须使用 R? + D? 标注。"

  must_not:
    - "不要输出多个直接 Target File。"
    - "不要把相关文件、阅读文件、依赖文件写成 Target File。"
    - "不要写 routing、next_actor、lock 或 status。"
    - "不要要求 KiroPrompt-GPT 写 current-state.json、current-input.md 或 durable-state.json。"
    - "不要请求未暴露的 Action。"
```

---

## 3. Required Sections

```yaml
required_sections:
  - "Task Header"
  - "Gateway Context Snapshot"
  - "Target File"
  - "File Context"
  - "Task Intent"
  - "Scope Boundary"
  - "Reading Scope"
  - "Expansion Permission"
  - "Expected Output"
  - "Prohibited Output"
```

---

## 4. Section Requirements

```yaml
section_requirements:
  Task_Header:
    purpose: "用极简方式标识本轮任务。"
    must_include:
      - "Task Title"
      - "Mode"
      - "Expected Action"
    must:
      - "Mode 必须来自 getLeaderContext.mode。"
      - "Expected Action 必须来自 getLeaderContext.expected_action。"
    must_not:
      - "不要根据聊天历史自行推断 mode。"
      - "不要自行改写 expected_action。"

  Gateway_Context_Snapshot:
    purpose: "给 KiroPrompt-GPT 提供必要的上游摘要，但不暴露 Gateway 内部实现。"
    must_include:
      - "Input Type"
      - "Current Input Meaning"
      - "Previous Target, if any"
      - "Durable / Last Cycle Notes, if useful"
    must:
      - "只写 KiroPrompt-GPT 需要理解任务的摘要。"
      - "rework 模式下必须说明返工来源和 previous_target。"
      - "received-summary 模式下必须说明上一轮已完成，可以继续下一轮。"
      - "first-round 模式下必须说明无上游 Summary / Rework。"
    must_not:
      - "不要写 bus、routing、events、full rounds history。"
      - "不要写其他 Agent 的内部状态。"
      - "不要要求 KiroPrompt-GPT 读取 Leader-GPT 私有状态。"

  Target_File:
    purpose: "定义本轮唯一直接目标文件。"
    must_include:
      - "Target File"
      - "File Operation"
      - "File Type"
    must:
      - "Target File 必须是单一路径。"
      - "Target File 必须与 leader_work_summary_json.file_change.target_file 一致。"
      - "File Operation 必须与 expected_action / file_change.type 语义一致。"
    file_operation_policy:
      when_expected_action_is_Create:
        should_use:
          - "create"
          - "continue-new-target"
      when_expected_action_is_MajorRevision:
        should_use:
          - "major-revision"
    must_not:
      - "不得把同一 Spec 的 design.md / requirements.md / tasks.md 同时写成 Target File。"
      - "不得把 Reading Scope 文件写成 Target File。"

  File_Context:
    purpose: "说明目标文件属于什么工程语义。"
    must_include:
      - "Belongs To"
      - "File Role"
      - "Upstream Basis"
    must_include_when_kiro_spec_file:
      - "Spec Name"
      - "Spec Layer: design.md | requirements.md | tasks.md"
      - "Spec Order: design.md → requirements.md → tasks.md"
      - "Upstream Spec Files"
    must_include_when_docs_file:
      - "Docs Module / Knowledge Unit"
      - "File Role"
      - "Related Modules"
    must:
      - "如果是 Kiro Spec 文件，必须说明当前层级。"
      - "如果是 tasks.md，必须说明其依赖 design.md 和 requirements.md。"
      - "如果是 requirements.md，必须说明其依赖 design.md。"
    must_not:
      - "不得把普通架构文稿与 Kiro Spec 三层文件混淆。"

  Task_Intent:
    purpose: "说明 Leader-GPT 希望 KiroPrompt-GPT 把什么意图转成后续执行提示。"
    should_include:
      - "本轮目标"
      - "选择该 target_file 的原因"
      - "该文件应解决的问题"
      - "该文件应推进到的层级"
      - "与当前项目阶段的关系"
      - "与 current_input_md / durable_state / last_cycle_state 的关系"
    mode_specific_requirements:
      first-round:
        must:
          - "说明这是第一轮任务选择。"
          - "说明没有上游 Summary 或 Rework。"
          - "说明目标来自 Instruction、Knowledge 和 durable_state。"
      received-summary:
        must:
          - "说明如何承接上一轮 Summary。"
          - "说明为什么本轮应推进该新目标。"
      rework:
        must:
          - "说明本轮是大修返工。"
          - "说明返工包要求。"
          - "说明 previous_target.target_file 与本轮目标的一致性。"
    must_not:
      - "不要要求 KiroPrompt-GPT 重新判断本轮 mode。"
      - "不要要求 KiroPrompt-GPT 决定 next_actor。"

  Scope_Boundary:
    purpose: "控制本轮范围，避免大杂烩。"
    must_include:
      - "In Scope"
      - "Out of Scope"
    must:
      - "In Scope 必须聚焦唯一 target_file。"
      - "Out of Scope 必须排除实现代码、无关模块、状态层写入和流程调度。"
      - "必须与 leader_work_summary_json.boundary 保持语义一致。"
    must_not:
      - "不要开放泛化重构。"
      - "不要扩展到其他 Agent Gateway。"
      - "不要进入 Gateway 内部实现。"

  Reading_Scope:
    purpose: "指定 KiroPrompt-GPT 需要读取的文件范围。"
    must:
      - "每个文件必须使用 R? + D? 标注。"
      - "每个文件必须带一句用途说明。"
      - "阅读范围只用于辅助生成 prompt，不等于 Target File。"
    must_not:
      - "不要写“相关文件”。"
      - "不要写“必要时参考”。"
      - "不要写“看一下模块文稿”。"
      - "不要列出完整项目文件树。"
      - "不要列 Leader-GPT 不可见或不应依赖的内部状态文件。"

  Expansion_Permission:
    purpose: "说明本轮是否允许 KiroPrompt-GPT 扩读。"
    must_include:
      - "Allow Expansion"
      - "Expansion Conditions"
      - "Max Expansion Times"
      - "Expansion Boundary"
    must:
      - "明确是否允许扩读。"
      - "明确触发条件。"
      - "明确最多扩读次数。"
      - "明确扩读不能改变 Target File。"
    must_not:
      - "不要写“视情况扩读”。"
      - "不要允许扩读 bus、routing、events 或其他 Agent 私有状态。"

  Expected_Output:
    purpose: "说明 KiroPrompt-GPT 应产出什么。"
    must_include:
      - "Output File"
      - "Output Requirements"
      - "Success Criteria"
    must:
      - "输出应是给 Kiro IDE 或下游执行者使用的提示内容。"
      - "输出必须聚焦唯一 target_file。"
      - "输出必须保留本轮边界。"
    must_not:
      - "不要要求 KiroPrompt-GPT 输出 Review-GPT 审查内容。"
      - "不要要求 KiroPrompt-GPT 输出 round meta。"
      - "不要要求 KiroPrompt-GPT 调用 Gateway Action。"

  Prohibited_Output:
    purpose: "列出本轮明确禁止 KiroPrompt-GPT 产出的内容。"
    must_include:
      - "不得写状态层。"
      - "不得写 routing / next_actor / lock / status。"
      - "不得请求额外上下文。"
      - "不得修改 Target File 之外的目标。"
```

---

## 5. Mode-Specific Rules

```yaml
mode_specific_rules:
  first-round:
    expected_action: "Create"
    task_packet_should:
      - "说明这是第一轮任务。"
      - "说明没有上游 Summary 或 Rework。"
      - "从项目方向、Knowledge 和 durable_state 中选择合理起点。"
    task_packet_must_not:
      - "假设存在 previous_target。"
      - "生成 MajorRevision 任务。"

  received-summary:
    expected_action: "Create"
    task_packet_should:
      - "吸收 current_input_md 中的 Summary。"
      - "承接上一轮 completed target。"
      - "选择下一轮新目标或继续推进下一个单文件任务。"
    task_packet_must_not:
      - "把 Summary 当作返工包。"
      - "重新审查上一轮完整流程。"
      - "生成 MajorRevision 任务。"

  rework:
    expected_action: "MajorRevision"
    task_packet_should:
      - "围绕 current_input_md 中的返工包生成大修任务。"
      - "优先使用 context.current_state.previous_target.target_file。"
      - "保持修改范围收敛。"
    task_packet_must_not:
      - "选择无关新 target_file。"
      - "生成 Create 任务。"
      - "忽略 previous_target。"
```

---

## 6. Suggested Markdown Skeleton

```markdown
<<<KIRO_PROMPT_TASK_PACKET_START>>>
# KiroPrompt-GPT Task Packet

## 1. Task Header
- Task Title:
- Mode:
- Input Type:
- Expected Action:

## 2. Gateway Context Snapshot
- Current Input Meaning:
- Previous Target:
- Durable State Notes:
- Last Cycle Notes:

## 3. Target File
- Target File:
- File Operation:
- File Type:

## 4. File Context
- Belongs To:
- File Role:
- Upstream Basis:
- Related Modules:

### If Kiro Spec File
- Spec Name:
- Spec Layer: design.md | requirements.md | tasks.md
- Spec Order: design.md → requirements.md → tasks.md
- Upstream Spec Files:

### If Docs / Knowledge File
- Docs Module / Knowledge Unit:
- File Role:
- Related Modules:

## 5. Task Intent
[说明本轮希望 KiroPrompt-GPT 生成什么方向的后续提示内容，以及为什么选择该 target_file。]

## 6. Scope Boundary

### In Scope
- ...

### Out of Scope
- ...

## 7. Reading Scope
- `...`: R0 + D5（用途说明）
- `...`: R1 + D4（用途说明）
- `...`: R2 + D3（用途说明）

## 8. Expansion Permission
- Allow Expansion:
- Expansion Conditions:
  - ...
- Max Expansion Times:
- Expansion Boundary:
  - ...

## 9. Expected Output
- Output File:
- Output Requirements:
  - ...
- Success Criteria:
  - ...

## 10. Prohibited Output
- 不得写状态层文件。
- 不得写 routing、next_actor、lock 或 status。
- 不得请求额外上下文。
- 不得改变本轮唯一 Target File。
- 不得输出 Review-GPT 的完整审查制度。
<<<KIRO_PROMPT_TASK_PACKET_END>>>
```

---

## 7. Pass / Fail Gate

```yaml
pass_when:
  - "使用 <<<KIRO_PROMPT_TASK_PACKET_START>>> 和 <<<KIRO_PROMPT_TASK_PACKET_END>>> 包裹。"
  - "包含所有 required_sections。"
  - "Mode / Input Type / Expected Action 来自 getLeaderContext。"
  - "Target File 是唯一直接目标文件。"
  - "Target File 与 leader_work_summary_json.file_change.target_file 一致。"
  - "Target File 与 review_intent_pack_md Review Target 一致。"
  - "Task Intent 与当前 mode 匹配。"
  - "Scope Boundary 与 leader_work_summary_json.boundary 语义一致。"
  - "Reading Scope 中每个文件都有 R? + D? 标注。"
  - "Expansion Permission 明确且有边界。"
  - "没有要求 KiroPrompt-GPT 写状态层或 routing。"

fail_when:
  - "缺少强 marker。"
  - "缺少唯一 Target File。"
  - "出现多个直接 Target File。"
  - "Target File 与 leader_work_summary_json.file_change.target_file 不一致。"
  - "Target File 与 review_intent_pack_md Review Target 不一致。"
  - "Reading Scope 出现未标注 R? + D? 的文件。"
  - "要求 KiroPrompt-GPT 写 current-state.json、current-input.md 或 durable-state.json。"
  - "要求 KiroPrompt-GPT 决定 routing、next_actor、lock 或 status。"
  - "请求未暴露 Action。"
  - "把聊天输出当作正式产物入口。"
```