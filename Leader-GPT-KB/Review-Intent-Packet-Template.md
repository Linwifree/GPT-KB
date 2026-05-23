# Review Intent Packet Template

用途：
本文件定义 Leader-GPT 生成 `commitLeaderArtifacts.review_intent_pack_md` 时必须遵守的 Markdown 结构、字段语义和边界规则。

`review_intent_pack_md` 是 Leader-GPT 提交给 Gateway 的三个产物之一。  
它不是聊天输出，不是独立文件写入口，而是 `commitLeaderArtifacts` requestBody 中的一个 Markdown string 字段。  
Gateway 接收后会将其保存为状态层中的 `review-intent-pack.md`，供后续 Review-GPT 使用。

---

## 1. Artifact Identity

```yaml
system_facts:
  file_identity:
    id: "review-intent-packet-template"
    name: "Review Intent Packet Template"
    owner_role: "Leader-GPT"
    use_stage: "before_commitLeaderArtifacts"
    purpose: "定义 commitLeaderArtifacts.review_intent_pack_md 的 Markdown 正文结构。"

  artifact_identity:
    action_field: "review_intent_pack_md"
    format: "Markdown string"
    submitted_via: "commitLeaderArtifacts"
    stored_by_gateway_as: "review-intent-pack.md"
    future_consumer: "Review-GPT"

  relation_to_gateway:
    - "Leader-GPT 每轮先调用 getLeaderContext。"
    - "Leader-GPT 根据 mode、input_type、expected_action 和 context 生成 review_intent_pack_md。"
    - "Leader-GPT 通过 commitLeaderArtifacts 一次性提交 task_packet_md、review_intent_pack_md、leader_work_summary_json。"
    - "review_intent_pack_md 必须与 leader_work_summary_json.file_change.target_file 保持一致。"

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

    does_not_own:
      - "不定义 KiroPrompt-GPT 的完整执行提示。"
      - "不定义 Review-GPT 的完整系统提示词。"
      - "不定义 leader_work_summary_json 的 schema。"
      - "不生成 round_id。"
      - "不生成 leader-output-meta.json。"
      - "不写 Gateway 内部状态。"
      - "不决定 routing、next_actor、lock 或 status。"
      - "不要求 Review-GPT 修改状态层。"
````

---

## 2. Required Wrapper

`review_intent_pack_md` 必须使用强 marker 包裹：

```text
<<<REVIEW_INTENT_PACKET_START>>>
# Review Intent Packet

...
<<<REVIEW_INTENT_PACKET_END>>>
```

```yaml
format_rules:
  format: "Markdown"
  wrapped_by:
    start: "<<<REVIEW_INTENT_PACKET_START>>>"
    end: "<<<REVIEW_INTENT_PACKET_END>>>"

  must:
    - "正文必须是非空 Markdown string。"
    - "必须包含唯一 Review Target。"
    - "Review Target 必须与 leader_work_summary_json.file_change.target_file 完全一致。"
    - "Review Target 必须与 task_packet_md 中的 Target File 完全一致。"
    - "如果 Optional Reading Scope 列文件，每个文件必须使用 R? + D? 标注。"

  must_not:
    - "不要输出多个直接 Review Target。"
    - "不要把相关文件、阅读文件、依赖文件写成 Review Target。"
    - "不要写 routing、next_actor、lock 或 status。"
    - "不要要求 Review-GPT 写 current-state.json、current-input.md 或 durable-state.json。"
    - "不要请求未暴露的 Action。"
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
    must:
      - "Mode 必须来自 getLeaderContext.mode。"
      - "Input Type 必须来自 getLeaderContext.input_type。"
      - "Expected Action 必须来自 getLeaderContext.expected_action。"
    must_not:
      - "不要根据聊天历史自行推断 mode。"
      - "不要自行改写 expected_action。"

  Review_Target:
    purpose: "说明本轮应审查的唯一 target_file，以及它所属的文件上下文。"
    must_include:
      - "Review Target"
      - "Related Spec / File Context"
      - "File Operation"
    must:
      - "Review Target 必须是单一目标文件。"
      - "Review Target 必须与 leader_work_summary_json.file_change.target_file 一致。"
      - "Review Target 必须与 task_packet_md Target File 一致。"
      - "rework 模式下 Review Target 应优先来自 context.current_state.previous_target.target_file。"
    must_not:
      - "不得默认把同一 Spec 的三层文件全部列为本轮审查目标。"
      - "不得把 Reading Scope 文件写成 Review Target。"

  Gateway_Context_Snapshot:
    purpose: "给 Review-GPT 提供必要的审查背景，但不暴露 Gateway 内部实现。"
    must_include:
      - "Current Input Meaning"
      - "Previous Target, if any"
      - "Relevant Summary or Rework Meaning"
      - "Durable / Last Cycle Notes, if useful"
    mode_specific:
      first-round:
        must:
          - "说明这是第一轮，没有上游 Summary 或 Rework。"
      received-summary:
        must:
          - "说明上一轮已完成，本轮是正常继续。"
          - "说明 Summary 只是下一轮选择依据，不是返工要求。"
      rework:
        must:
          - "说明本轮是大修返工。"
          - "说明返工包是审查依据。"
          - "说明 previous_target 是主要审查目标。"
    must_not:
      - "不要写 bus、routing、events、full rounds history。"
      - "不要写其他 Agent 的内部状态。"
      - "不要要求 Review-GPT 依赖 Leader-GPT 私有状态。"

  Leader_Intent:
    purpose: "说明 Leader-GPT 本轮原本希望产物完成什么目标。"
    should_include:
      - "本轮生成目标"
      - "选择该 target_file 的原因"
      - "产物应解决的问题"
      - "产物不应扩展到的范围"
      - "本轮与 mode / expected_action 的关系"
    must:
      - "必须与 task_packet_md 的 Task Intent 保持语义一致。"
      - "必须与 leader_work_summary_json.main_goal 保持语义一致。"

  Expected_Alignment:
    purpose: "说明 Review-GPT 审查时应检查产物是否与哪些方向保持一致。"
    should_include:
      - "当前项目阶段"
      - "对应模块架构边界"
      - "Kiro Specs 的 design.md → requirements.md → tasks.md 顺序"
      - "单一 target_file 边界"
      - "低风险 / 不进入广泛实现的当前阶段约束，如本轮相关"
      - "current_input_md 中 Summary 或 Rework 的要求"
    must_not:
      - "不要要求 Review-GPT 审查本轮之外的未来系统设计。"
      - "不要要求 Review-GPT 追溯完整自动化链路。"

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
        should:
          - "审查第一轮目标选择是否合理。"
          - "审查是否错误依赖不存在的 Summary / Rework。"
      received-summary:
        should:
          - "审查是否自然承接上一轮 Summary。"
          - "审查是否错误将 Summary 当作返工。"
      rework:
        should:
          - "审查是否围绕返工包和 previous_target。"
          - "审查是否解决返工要求。"
          - "审查是否没有扩大大修范围。"

  Do_Not_Overreview:
    purpose: "防止 Review-GPT 过度审查。"
    must_include:
      - "不要把 tasks.md 审成 PR 计划。"
      - "不要要求改变 Kiro 原生任务块格式。"
      - "不要因为没有实现代码而判定 Spec 失败。"
      - "不要审查本轮目标之外的内容。"
      - "不要要求补齐未被本轮 Target File 覆盖的其他文件。"
      - "不要要求 Review-GPT 决定 routing 或 next_actor。"
    must_not:
      - "不要把 Review-GPT 变成总调度器。"
      - "不要把 Review-GPT 变成 KiroPrompt-GPT。"

  Optional_Reading_Scope:
    purpose: "仅在 Review-GPT 需要按需阅读时使用。"
    may:
      - "可以为空。"
      - "可以写无。"
      - "可以列少量审查所需文件。"
    must:
      - "如果列文件，必须使用 R? + D? 标注。"
      - "每个文件必须有用途说明。"
    must_not:
      - "不要列完整项目文件树。"
      - "不要列 Leader-GPT 不可见或不应依赖的内部状态文件。"
      - "不要要求读取 bus、routing、events 或 full rounds history。"

  Review_Result_Expectation:
    purpose: "说明 Review-GPT 的输出应如何被下游理解。"
    should_include:
      - "通过条件"
      - "小修条件"
      - "大修条件"
    must:
      - "通过：目标完成，边界正确，无需修改。"
      - "小修：存在局部文本/结构问题，可由补丁修正。"
      - "大修：目标偏离、边界错误、mode/action 语义冲突或返工要求未解决。"
    must_not:
      - "不要要求 Review-GPT 直接写状态层。"
      - "不要要求 Review-GPT 直接触发 Codex 或 Kiro。"

  Prohibited_Review_Behavior:
    purpose: "列出 Review-GPT 不应做的事。"
    must_include:
      - "不得审查 Gateway 内部实现。"
      - "不得要求读取未暴露状态。"
      - "不得决定 next_actor。"
      - "不得修改状态层。"
      - "不得把聊天输出当作状态层产物。"
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
    review_intent_must_not:
      - "要求 Review-GPT 寻找上一轮 Summary。"
      - "要求 Review-GPT 寻找返工包。"
      - "将本轮误判为 MajorRevision。"

  received-summary:
    expected_action: "Create"
    review_intent_should:
      - "审查本轮是否自然承接上一轮 Summary。"
      - "审查是否推进新的下一轮目标。"
      - "审查是否避免重复上一轮已完成内容。"
    review_intent_must_not:
      - "把 Summary 审成返工包。"
      - "要求对上一轮 completed target 做大修。"
      - "将本轮误判为 MajorRevision。"

  rework:
    expected_action: "MajorRevision"
    review_intent_should:
      - "审查大修任务是否围绕返工包。"
      - "审查 target_file 是否优先使用 previous_target.target_file。"
      - "审查是否解决返工包指出的问题。"
      - "审查是否没有扩大返工范围。"
    review_intent_must_not:
      - "选择无关新目标。"
      - "将返工包当作普通 Summary。"
      - "将本轮误判为 Create。"
```

---

## 6. Suggested Markdown Skeleton

```markdown
<<<REVIEW_INTENT_PACKET_START>>>
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
- 不要要求 Review-GPT 决定 routing 或 next_actor。

## 8. Optional Reading Scope
- `...`: R0 + D5（用途说明）
- `...`: R1 + D3（用途说明）

## 9. Review Result Expectation
- PASS:
- SMALL_FIX:
- MAJOR_REVISION:

## 10. Prohibited Review Behavior
- 不得审查 Gateway 内部实现。
- 不得要求读取 bus、routing、events 或 full rounds history。
- 不得写 current-state.json、current-input.md 或 durable-state.json。
- 不得决定 next_actor、routing、lock 或 status。
<<<REVIEW_INTENT_PACKET_END>>>
```

---

## 7. Pass / Fail Gate

```yaml
pass_when:
  - "使用 <<<REVIEW_INTENT_PACKET_START>>> 和 <<<REVIEW_INTENT_PACKET_END>>> 包裹。"
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
  - "没有要求 Review-GPT 写状态层或 routing。"

fail_when:
  - "缺少强 marker。"
  - "缺少唯一 Review Target。"
  - "出现多个直接 Review Target。"
  - "Review Target 与 leader_work_summary_json.file_change.target_file 不一致。"
  - "Review Target 与 task_packet_md Target File 不一致。"
  - "Optional Reading Scope 出现未标注 R? + D? 的文件。"
  - "要求 Review-GPT 写 current-state.json、current-input.md 或 durable-state.json。"
  - "要求 Review-GPT 决定 routing、next_actor、lock 或 status。"
  - "要求 Review-GPT 请求未暴露 Action。"
  - "把 Review-GPT 变成总调度器。"
  - "把聊天输出当作正式产物入口。"
```
