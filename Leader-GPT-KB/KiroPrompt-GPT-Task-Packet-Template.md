# KiroPrompt-GPT Task Packet Template

用途：
本文件定义 Leader-GPT 生成 `task_packet_md` 的 Markdown 正文时应遵守的章节结构、字段语义和任务边界。

`task_packet_md` 是给 KiroPrompt-GPT 使用的任务包正文。  
本文件只定义 Markdown 正文内容，不定义 `commitLeaderArtifacts` requestBody 的字段形态。  
提交体字段形态以 `Leader-GPT-Commit-Payload-Format.yaml` 为准。

---

## 1. Artifact Identity

```yaml
system_facts:
  file_identity:
    id: "kiro-prompt-gpt-task-packet-template"
    name: "KiroPrompt-GPT Task Packet Template"
    owner_role: "Leader-GPT"
    use_stage: "build_task_packet_md"
    purpose: "定义 task_packet_md 的 Markdown 正文结构。"

  artifact_identity:
    action_field: "task_packet_md"
    format: "Markdown document"
    future_consumer: "KiroPrompt-GPT"

  gateway_schema_alignment:
    field_type: "string"
    minLength: 1
    cleaned_value: "non_empty"

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

    out_of_scope:
      - "Review-GPT 审查制度。"
      - "leader_work_summary_json schema。"
      - "提交体字段形态。"
```

---

## 2. Markdown Document Shape

```yaml
document_shape:
  title:
    h1: "# KiroPrompt-GPT Task Packet"

  required_sections:
    - "## 1. Task Header"
    - "## 2. Gateway Context Snapshot"
    - "## 3. Target File"
    - "## 4. File Context"
    - "## 5. Task Intent"
    - "## 6. Scope Boundary"
    - "## 7. Reading Scope"
    - "## 8. Expansion Permission"
    - "## 9. Expected Output"
    - "## 10. Prohibited Output"

  gateway_checked_shape:
    - "task_packet_md 字段存在。"
    - "task_packet_md 是 string。"
    - "清洗后 task_packet_md 非空。"

  template_quality_shape:
    - "正文聚焦给 KiroPrompt-GPT 的任务包。"
    - "正文围绕本轮唯一 Target File。"
    - "正文中的 Target File 与 leader_work_summary_json.file_change.target_file 保持一致。"
    - "正文中的 Target File 与 review_intent_pack_md 中的 Review Target 保持一致。"
    - "Reading Scope 中每个文件使用 R? + D? 标注。"
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
      - "Input Type"
      - "Expected Action"
    required_alignment:
      - "Mode 来自 getLeaderContext.mode。"
      - "Input Type 来自 getLeaderContext.input_type。"
      - "Expected Action 来自 getLeaderContext.expected_action。"

  Gateway_Context_Snapshot:
    purpose: "给 KiroPrompt-GPT 提供必要的上游摘要。"
    must_include:
      - "Input Type"
      - "Current Input Meaning"
      - "Previous Target, if any"
      - "Durable / Last Cycle Notes, if useful"
    mode_notes:
      first-round:
        - "说明这是第一轮。"
        - "说明没有上游 Summary 或 Rework。"
      received-summary:
        - "说明上一轮已完成。"
        - "说明本轮可以继续下一轮。"
      rework:
        - "说明本轮是大修返工。"
        - "说明返工来源和 previous_target。"

  Target_File:
    purpose: "定义本轮唯一直接目标文件。"
    must_include:
      - "Target File"
      - "File Operation"
      - "File Type"
    required_alignment:
      - "Target File 是单一路径。"
      - "Target File 使用本轮 canonical target_file。"
      - "Target File 与 leader_work_summary_json.file_change.target_file 一致。"
      - "File Operation 与 expected_action / file_change.type 语义一致。"
    file_operation_policy:
      when_expected_action_is_Create:
        suggested_values:
          - "create"
          - "continue-new-target"
      when_expected_action_is_MajorRevision:
        suggested_values:
          - "major-revision"

  File_Context:
    purpose: "说明目标文件属于什么工程语义。"
    must_include:
      - "Belongs To"
      - "File Role"
      - "Upstream Basis"
    when_kiro_spec_file:
      must_include:
        - "Spec Name"
        - "Spec Layer: design.md | requirements.md | tasks.md"
        - "Spec Order: design.md → requirements.md → tasks.md"
        - "Upstream Spec Files"
    when_docs_file:
      must_include:
        - "Docs Module / Knowledge Unit"
        - "File Role"
        - "Related Modules"
    required_alignment:
      - "Kiro Spec 文件需要说明当前层级。"
      - "requirements.md 需要说明其依赖 design.md。"
      - "tasks.md 需要说明其依赖 design.md 和 requirements.md。"
      - "普通架构文稿与 Kiro Spec 三层文件保持区分。"

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
        - "说明这是第一轮任务选择。"
        - "说明目标来自 Instruction、Knowledge 和 durable_state。"
      received-summary:
        - "说明如何承接上一轮 Summary。"
        - "说明为什么本轮应推进该新目标。"
      rework:
        - "说明本轮是大修返工。"
        - "说明返工包要求。"
        - "说明 previous_target.target_file 与本轮目标的一致性。"

  Scope_Boundary:
    purpose: "控制本轮范围，避免大杂烩。"
    must_include:
      - "In Scope"
      - "Out of Scope"
    required_alignment:
      - "In Scope 聚焦唯一 target_file。"
      - "Out of Scope 排除实现代码、无关模块和非本轮目标。"
      - "Scope Boundary 与 leader_work_summary_json.boundary 保持语义一致。"

  Reading_Scope:
    purpose: "指定 KiroPrompt-GPT 需要读取的文件范围。"
    required_alignment:
      - "每个文件使用 R? + D? 标注。"
      - "每个文件带一句用途说明。"
      - "阅读范围只用于辅助生成 prompt，不等于 Target File。"

  Expansion_Permission:
    purpose: "说明本轮是否允许 KiroPrompt-GPT 扩读。"
    must_include:
      - "Allow Expansion"
      - "Expansion Conditions"
      - "Max Expansion Times"
      - "Expansion Boundary"
    required_alignment:
      - "明确是否允许扩读。"
      - "明确触发条件。"
      - "明确最多扩读次数。"
      - "明确扩读不改变 Target File。"

  Expected_Output:
    purpose: "说明 KiroPrompt-GPT 应产出什么。"
    must_include:
      - "Output File"
      - "Output Requirements"
      - "Success Criteria"
    required_alignment:
      - "输出面向 Kiro IDE 或下游执行者使用。"
      - "输出聚焦唯一 target_file。"
      - "输出保留本轮边界。"

  Prohibited_Output:
    purpose: "列出本轮不属于 KiroPrompt-GPT 任务包的内容。"
    should_include:
      - "非本轮 Target File。"
      - "Review-GPT 审查制度。"
      - "leader_work_summary_json 内容。"
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
    expected_shape:
      - "action = Create。"
      - "file_change.type = new_file。"

  received-summary:
    expected_action: "Create"
    task_packet_should:
      - "吸收 current_input_md 中的 Summary。"
      - "承接上一轮 completed target。"
      - "选择下一轮新目标或继续推进下一个单文件任务。"
    expected_shape:
      - "action = Create。"
      - "file_change.type = new_file。"

  rework:
    expected_action: "MajorRevision"
    task_packet_should:
      - "围绕 current_input_md 中的返工包生成大修任务。"
      - "优先使用 context.current_state.previous_target.target_file。"
      - "保持修改范围收敛。"
    expected_shape:
      - "action = MajorRevision。"
      - "file_change.type = major_revision。"
```

---

## 6. Suggested Markdown Skeleton

```markdown
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
- 非本轮 Target File。
- Review-GPT 审查制度。
- leader_work_summary_json 内容。
```

---

## 7. Template Quality Gate

```yaml
gateway_schema_pass_when:
  - "task_packet_md 字段存在。"
  - "task_packet_md 是 string。"
  - "清洗后 task_packet_md 非空。"

template_quality_pass_when:
  - "task_packet_md 使用本模板定义的 Markdown 正文结构。"
  - "包含所有 required_sections。"
  - "Mode / Input Type / Expected Action 来自 getLeaderContext。"
  - "Target File 是唯一直接目标文件。"
  - "Target File 与 leader_work_summary_json.file_change.target_file 一致。"
  - "Target File 与 review_intent_pack_md Review Target 一致。"
  - "Task Intent 与当前 mode 匹配。"
  - "Scope Boundary 与 leader_work_summary_json.boundary 语义一致。"
  - "Reading Scope 中每个文件都有 R? + D? 标注。"
  - "Expansion Permission 明确且有边界。"
  - "正文聚焦给 KiroPrompt-GPT 的任务包。"

repair_focus:
  - "补齐 Target File。"
  - "统一 Target File 与 canonical target_file。"
  - "补齐 Reading Scope 的 R? + D? 标注。"
  - "收束正文到 KiroPrompt-GPT 任务包。"
```