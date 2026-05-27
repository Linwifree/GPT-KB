# Review Intent Packet Template

用途：
本文件定义 Leader-GPT 生成 `review_intent_pack_md` 的 Markdown 正文时必须遵守的章节结构、字段语义和边界规则。

`review_intent_pack_md` 是给 Review-GPT 使用的审查意图正文。  
本文件只定义 Markdown 正文内容，不定义 `commitLeaderArtifacts` requestBody 的字段形态。  
提交体字段形态以 `Leader-GPT-Commit-Payload-Format.yaml` 为准。

---

## 1. Artifact Identity

```yaml
system_facts:
  file_identity:
    id: "review-intent-packet-template"
    name: "Review Intent Packet Template"
    owner_role: "Leader-GPT"
    use_stage: "build_review_intent_pack_md"
    purpose: "定义 review_intent_pack_md 的 Markdown 正文结构。"

  artifact_identity:
    action_field: "review_intent_pack_md"
    format: "Markdown document"
    future_consumer: "Review-GPT"

  scope_boundary:
    owns:
      - "本轮审查目标。"
      - "唯一 Review Target。"
      - "Leader Intent。"
      - "Expected Alignment。"
      - "Review Focus。"
      - "Do Not Overreview。"
      - "Optional Reading Scope。"
      - "Review Result Expectation。"

    out_of_scope:
      - "KiroPrompt-GPT 的完整执行提示。"
      - "Review-GPT 的完整系统提示词。"
      - "leader_work_summary_json schema。"
      - "提交体字段形态。"
      - "状态层元信息。"
      - "Gateway 实现细节。"
```

---

## 2. Markdown Document Shape

```yaml
document_shape:
  title:
    h1: "# Review Intent Packet"

  required_sections:
    - "## 1. Review Header"
    - "## 2. Review Target"
    - "## 3. Gateway Context Snapshot"
    - "## 4. Leader Intent"
    - "## 5. Expected Alignment"
    - "## 6. Review Focus"
    - "## 7. Do Not Overreview"
    - "## 8. Optional Reading Scope"
    - "## 9. Review Result Expectation"
    - "## 10. Prohibited Review Behavior"

  content_policy:
    - "正文聚焦给 Review-GPT 的审查意图。"
    - "正文围绕本轮唯一 Review Target。"
    - "正文中的 Review Target 与 leader_work_summary_json.file_change.target_file 保持一致。"
    - "正文中的 Review Target 与 task_packet_md 中的 Target File 保持一致。"
    - "Optional Reading Scope 如列文件，每个文件使用 R? + D? 标注。"
```

---

## 3. Required Sections

```yaml
required_sections:
  - "Review Header"
  - "Review Target"
  - "Gateway Context Snapshot"
  - "Leader Intent"
  - "Expected Alignment"
  - "Review Focus"
  - "Do Not Overreview"
  - "Optional Reading Scope"
  - "Review Result Expectation"
  - "Prohibited Review Behavior"
```

---

## 4. Section Requirements

```yaml
section_requirements:
  Review_Header:
    purpose: "用极简方式标识本轮审查意图。"
    must_include:
      - "Mode"
      - "Input Type"
      - "Expected Action"
    required_alignment:
      - "Mode 来自 getLeaderContext.mode。"
      - "Input Type 来自 getLeaderContext.input_type。"
      - "Expected Action 来自 getLeaderContext.expected_action。"

  Review_Target:
    purpose: "说明本轮应审查的唯一 target_file，以及它所属的文件上下文。"
    must_include:
      - "Review Target"
      - "Related Spec / File Context"
      - "File Operation"
    required_alignment:
      - "Review Target 是单一目标文件。"
      - "Review Target 与 leader_work_summary_json.file_change.target_file 一致。"
      - "Review Target 与 task_packet_md Target File 一致。"
      - "rework 模式下 Review Target 优先来自 context.current_state.previous_target.target_file。"

  Gateway_Context_Snapshot:
    purpose: "给 Review-GPT 提供必要的审查背景。"
    must_include:
      - "Current Input Meaning"
      - "Previous Target, if any"
      - "Relevant Summary or Rework Meaning"
      - "Durable / Last Cycle Notes, if useful"
    mode_specific:
      first-round:
        - "说明这是第一轮，没有上游 Summary 或 Rework。"
      received-summary:
        - "说明上一轮已完成，本轮是正常继续。"
        - "说明 Summary 只是下一轮选择依据，不是返工要求。"
      rework:
        - "说明本轮是大修返工。"
        - "说明返工包是审查依据。"
        - "说明 previous_target 是主要审查目标。"

  Leader_Intent:
    purpose: "说明 Leader-GPT 本轮原本希望产物完成什么目标。"
    should_include:
      - "本轮生成目标"
      - "选择该 target_file 的原因"
      - "产物应解决的问题"
      - "产物不应扩展到的范围"
      - "本轮与 mode / expected_action 的关系"
    required_alignment:
      - "与 task_packet_md 的 Task Intent 保持语义一致。"
      - "与 leader_work_summary_json.main_goal 保持语义一致。"

  Expected_Alignment:
    purpose: "说明 Review-GPT 审查时应检查产物是否与哪些方向保持一致。"
    should_include:
      - "当前项目阶段"
      - "对应模块架构边界"
      - "Kiro Specs 的 design.md → requirements.md → tasks.md 顺序"
      - "单一 target_file 边界"
      - "低风险 / 不进入广泛实现的当前阶段约束，如本轮相关"
      - "current_input_md 中 Summary 或 Rework 的要求"

  Review_Focus:
    purpose: "说明本轮审查重点。"
    should_include:
      - "是否完成本轮目标"
      - "是否保持单一 target_file"
      - "是否保持单一工程域"
      - "是否存在跨模块大杂烩"
      - "是否与已有模块边界冲突"
      - "是否与 Spec 层级语义一致"
      - "是否遵守 mode 对应的 action 语义"
    mode_specific:
      first-round:
        - "审查第一轮目标选择是否合理。"
        - "审查是否错误依赖不存在的 Summary / Rework。"
      received-summary:
        - "审查是否自然承接上一轮 Summary。"
        - "审查是否错误将 Summary 当作返工。"
      rework:
        - "审查是否围绕返工包和 previous_target。"
        - "审查是否解决返工要求。"
        - "审查是否没有扩大大修范围。"

  Do_Not_Overreview:
    purpose: "防止 Review-GPT 过度审查。"
    should_include:
      - "不要把 tasks.md 审成 PR 计划。"
      - "不要要求改变 Kiro 原生任务块格式。"
      - "不要因为没有实现代码而判定 Spec 失败。"
      - "不要审查本轮目标之外的内容。"
      - "不要要求补齐未被本轮 Target File 覆盖的其他文件。"

  Optional_Reading_Scope:
    purpose: "仅在 Review-GPT 需要按需阅读时使用。"
    allowed_shape:
      - "可以为空。"
      - "可以写无。"
      - "可以列少量审查所需文件。"
    required_alignment:
      - "如果列文件，使用 R? + D? 标注。"
      - "每个文件有用途说明。"

  Review_Result_Expectation:
    purpose: "说明 Review-GPT 的输出应如何被下游理解。"
    should_include:
      - "通过条件"
      - "小修条件"
      - "大修条件"
    expected_meaning:
      PASS: "目标完成，边界正确，无需修改。"
      SMALL_FIX: "存在局部文本/结构问题，可由补丁修正。"
      MAJOR_REVISION: "目标偏离、边界错误、mode/action 语义冲突或返工要求未解决。"

  Prohibited_Review_Behavior:
    purpose: "列出本轮不属于 Review-GPT 审查意图的内容。"
    should_include:
      - "Gateway 实现细节。"
      - "状态层写入。"
      - "非本轮 Review Target。"
      - "KiroPrompt-GPT 完整执行提示。"
```

---

## 5. Mode-Specific Rules

```yaml
mode_specific_rules:
  first-round:
    expected_action: "Create"
    review_intent_should:
      - "审查第一轮目标是否适合作为起点。"
      - "审查是否基于 Instruction、Knowledge、durable_state，而不是虚构 Summary。"
      - "审查是否保持单一 target_file。"
    expected_shape:
      - "action = Create。"
      - "file_change.type = new_file。"

  received-summary:
    expected_action: "Create"
    review_intent_should:
      - "审查本轮是否自然承接上一轮 Summary。"
      - "审查是否推进新的下一轮目标。"
      - "审查是否避免重复上一轮已完成内容。"
    expected_shape:
      - "action = Create。"
      - "file_change.type = new_file。"

  rework:
    expected_action: "MajorRevision"
    review_intent_should:
      - "审查大修任务是否围绕返工包。"
      - "审查 target_file 是否优先使用 previous_target.target_file。"
      - "审查是否解决返工包指出的问题。"
      - "审查是否没有扩大返工范围。"
    expected_shape:
      - "action = MajorRevision。"
      - "file_change.type = major_revision。"
```

---

## 6. Suggested Markdown Skeleton

```markdown
# Review Intent Packet

## 1. Review Header
- Mode:
- Input Type:
- Expected Action:

## 2. Review Target
- Review Target:
- File Operation:
- Related Spec / File Context:
- Target Source:

## 3. Gateway Context Snapshot
- Current Input Meaning:
- Previous Target:
- Summary / Rework Meaning:
- Durable State Notes:
- Last Cycle Notes:

## 4. Leader Intent
[说明 Leader-GPT 本轮原本希望产物完成什么目标。]

## 5. Expected Alignment
- ...
- ...

## 6. Review Focus
- ...
- ...

## 7. Do Not Overreview
- 不要把 tasks.md 审成 PR 计划。
- 不要要求改变 Kiro 原生任务块格式。
- 不要因为没有实现代码而判定 Spec 失败。
- 不要审查本轮目标之外的内容。
- 不要要求补齐未被本轮 Target File 覆盖的其他文件。

## 8. Optional Reading Scope
- `...`: R0 + D5（用途说明）
- `...`: R1 + D3（用途说明）

## 9. Review Result Expectation
- PASS:
- SMALL_FIX:
- MAJOR_REVISION:

## 10. Prohibited Review Behavior
- Gateway 实现细节。
- 状态层写入。
- 非本轮 Review Target。
- KiroPrompt-GPT 完整执行提示。
```

---

## 7. Pass / Fail Gate

```yaml
pass_when:
  - "review_intent_pack_md 使用本模板定义的 Markdown 正文结构。"
  - "包含所有 required_sections。"
  - "Mode / Input Type / Expected Action 来自 getLeaderContext。"
  - "Review Target 是唯一直接审查目标。"
  - "Review Target 与 leader_work_summary_json.file_change.target_file 一致。"
  - "Review Target 与 task_packet_md Target File 一致。"
  - "Leader Intent 与 task_packet_md 的 Task Intent 语义一致。"
  - "Expected Alignment 与当前 mode 和项目阶段一致。"
  - "Review Focus 聚焦本轮 target_file。"
  - "Do Not Overreview 明确防止过度审查。"
  - "Optional Reading Scope 如列文件则全部有 R? + D? 标注。"
  - "正文聚焦给 Review-GPT 的审查意图。"

fail_when:
  - "缺少唯一 Review Target。"
  - "出现多个直接 Review Target。"
  - "Review Target 与 leader_work_summary_json.file_change.target_file 不一致。"
  - "Review Target 与 task_packet_md Target File 不一致。"
  - "Optional Reading Scope 出现未标注 R? + D? 的文件。"
  - "正文混入 KiroPrompt-GPT 完整执行提示。"
  - "正文混入 leader_work_summary_json 内容。"
  - "正文混入 Gateway 实现细节。"
```