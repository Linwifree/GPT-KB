# Leader-GPT Actions Gateway Contract

用途：
本文件定义 Leader-GPT 在 Actions 模式下能感知到的 Gateway 交互边界。

本文件只描述 Leader-GPT 可见的 Actions、输入字段、输出字段、提交体 schema、运行时校验结果和错误处理方式。  
本文件不解释 Gateway 内部实现，不描述其他 Agent 的内部状态，不承担 target_file 选择策略。

---

## 1. Leader-GPT 与 Gateway 的关系

```yaml
system_facts:
  file_identity:
    id: "leader-gpt-actions-gateway-contract"
    name: "Leader-GPT Actions Gateway Contract"
    owner_role: "Leader-GPT"
    use_stage: "actions_mode"

  leader_visible_world:
    - "Leader-GPT 通过 Actions 与 Gateway 交互。"
    - "Leader-GPT 每轮先调用 getLeaderContext 读取本轮允许上下文。"
    - "Leader-GPT 根据 Gateway 返回的 mode、input_type、expected_action 和 context 生成三个产物。"
    - "Leader-GPT 通过 commitLeaderArtifacts 一次性提交三个产物。"
    - "Gateway accepted=true 后，Leader-GPT 在聊天中只输出极简提交结果。"

  visible_boundary:
    can_use:
      - "getLeaderContext"
      - "commitLeaderArtifacts"
    can_see:
      - "getLeaderContext 返回的上下文。"
      - "commitLeaderArtifacts 返回的提交结果。"
    works_through:
      - "Gateway Actions"
```

---

## 2. Source Contracts

```yaml
source_contracts:
  commit_payload_format:
    file: "Leader-GPT-Commit-Payload-Format.yaml"
    role: "定义 commitLeaderArtifacts requestBody 的整体 schema。"

  work_summary_template:
    file: "Leader-Work-Summary-Template.yaml"
    role: "定义 leader_work_summary_json 的严格 LeaderWorkSummary schema。"

  task_packet_template:
    file: "KiroPrompt-GPT-Task-Packet-Template.md"
    role: "定义 task_packet_md 的 Markdown 正文结构。"

  review_intent_template:
    file: "Review-Intent-Packet-Template.md"
    role: "定义 review_intent_pack_md 的 Markdown 正文结构。"

  repository_path_map:
    file: "Repository-Path-Map.yaml"
    role: "定义共享路径族、WSL/POSIX 路径写法和 targetable 路径族。"

  target_file_selection_policy:
    file: "Leader-GPT-Target-File-Selection-Policy.yaml"
    role: "定义 Leader-GPT 如何选择一轮唯一 canonical target_file。"
```

---

## 3. Leader-GPT 可用 Actions

```yaml
available_actions:
  getLeaderContext:
    operationId: "getLeaderContext"
    method: "GET"
    path: "/api/projects/{projectId}/leader/context"
    purpose:
      - "读取本轮 Leader-GPT 被允许看到的上下文。"
      - "获得 mode、input_type、expected_action。"
      - "获得 context.current_state、context.current_input_md、context.optional_context。"

  commitLeaderArtifacts:
    operationId: "commitLeaderArtifacts"
    method: "POST"
    path: "/api/projects/{projectId}/leader/artifacts"
    purpose:
      - "提交本轮三个规划产物。"
      - "提交 task_packet_md。"
      - "提交 review_intent_pack_md。"
      - "提交 leader_work_summary_json。"

action_boundary:
  exposed_actions:
    - "getLeaderContext"
    - "commitLeaderArtifacts"
  request_context_source:
    - "getLeaderContext"
  artifact_submission_entry:
    - "commitLeaderArtifacts"
```

---

## 4. 每轮标准调用顺序

```yaml
standard_action_flow:
  - step: 1
    action: "调用 getLeaderContext(projectId)。"

  - step: 2
    action: "读取 Gateway 返回值。"
    read_fields:
      - "accepted"
      - "project_id"
      - "mode"
      - "input_type"
      - "expected_action"
      - "context.current_state"
      - "context.current_input_md"
      - "context.optional_context.last_cycle_state"
      - "context.optional_context.durable_state"

  - step: 3
    action: "根据 mode / input_type / expected_action 判断本轮任务类型。"

  - step: 4
    action: "使用 Repository-Path-Map.yaml 与 Leader-GPT-Target-File-Selection-Policy.yaml 确定唯一 canonical target_file。"

  - step: 5
    action: "生成三个产物。"
    outputs:
      - "task_packet_md"
      - "review_intent_pack_md"
      - "leader_work_summary_json"

  - step: 6
    action: "使用 Leader-GPT-Actions-Commit-Gate.yaml 做提交前自检。"

  - step: 7
    action: "调用 commitLeaderArtifacts(projectId, requestBody)。"

  - step: 8
    action: "根据 Gateway 响应返回极简聊天结果。"

hard_rules:
  required:
    - "每轮开始先调用 getLeaderContext。"
    - "mode、input_type、expected_action 来自 getLeaderContext 返回值。"
    - "leader_work_summary_json.action 等于 expected_action。"
    - "canonical target_file 同步到三个产物。"
    - "最终有效产物通过 commitLeaderArtifacts 提交。"
    - "commitLeaderArtifacts 成功后，聊天中只返回极简回执。"
```

---

## 5. getLeaderContext 返回结构

```yaml
getLeaderContext_response:
  accepted:
    type: "boolean"
    expected: true
    meaning: "Gateway 是否接受本次上下文读取请求。"

  project_id:
    type: "string"
    meaning: "当前项目 ID。"

  mode:
    type: "enum string"
    allowed:
      - "first-round"
      - "received-summary"
      - "rework"
    meaning: "Gateway 为本轮解析出的执行模式。"

  input_type:
    type: "enum string"
    allowed:
      - "first-round"
      - "received-summary"
      - "rework"
    meaning: "Gateway 从 current_input_md 中解析出的输入类型。"

  expected_action:
    type: "enum string"
    allowed:
      - "Create"
      - "MajorRevision"
    meaning: "Leader-GPT 本轮 leader_work_summary_json.action 必须使用的值。"

  context:
    type: "object"
    contains:
      - "current_state"
      - "current_input_md"
      - "optional_context"
```

---

## 6. mode / input_type / expected_action 规则

```yaml
mode_policy:
  source_of_truth:
    - "mode 来自 getLeaderContext。"
    - "input_type 来自 getLeaderContext。"
    - "expected_action 来自 getLeaderContext。"

  first-round:
    input_type: "first-round"
    expected_action: "Create"
    meaning:
      - "第一轮，没有上游 Summary 或返工包。"
      - "Leader-GPT 根据 Instruction、Knowledge、durable_state 和当前项目方向选择第一轮任务。"

  received-summary:
    input_type: "received-summary"
    expected_action: "Create"
    meaning:
      - "上一轮已被接收。"
      - "Leader-GPT 基于 current_input_md 中的 Summary 继续下一轮。"

  rework:
    input_type: "rework"
    expected_action: "MajorRevision"
    meaning:
      - "当前轮次是大修返工。"
      - "Leader-GPT 围绕返工包和 previous_target 生成大修任务。"

action_rule:
  required:
    - "leader_work_summary_json.action 等于 expected_action。"
    - "first-round / received-summary 对应 Create。"
    - "rework 对应 MajorRevision。"
```

---

## 7. context.current_state

```yaml
current_state:
  type: "object"
  meaning:
    - "Gateway 裁剪后允许 Leader-GPT 看到的当前状态。"
    - "只作为本轮任务判断的可见上下文。"

  may_contain:
    - "action_reminder"
    - "previous_target"
    - "required_inputs"
    - "optional_context"
    - "must_output"

  first_round_meaning:
    previous_target: null
    behavior:
      - "选择第一轮任务。"
      - "不等待 Summary 或 Rework。"

  received_summary_meaning:
    previous_target:
      meaning: "上一轮完成的目标文件。"
      expected_state: "completed"
    behavior:
      - "基于 Summary 选择下一轮任务。"

  rework_meaning:
    previous_target:
      meaning: "需要大修返工的目标文件。"
      expected_state: "rework"
    behavior:
      - "优先围绕 previous_target.target_file 生成大修任务。"
```

---

## 8. context.current_input_md

```yaml
current_input_md:
  type: "Markdown string"
  meaning:
    - "本轮真正的输入正文。"
    - "由 Gateway 读取后返回给 Leader-GPT。"

  first_round:
    meaning:
      - "说明当前是第一轮。"
      - "没有上游 Summary。"
      - "没有返工包。"

  received_summary:
    meaning:
      - "承载上一轮已接收 Summary。"
      - "Leader-GPT 据此选择下一轮任务。"

  rework:
    meaning:
      - "承载 Review-GPT 返工包。"
      - "Leader-GPT 围绕返工目标生成 MajorRevision 任务。"
```

---

## 9. context.optional_context

```yaml
optional_context:
  last_cycle_state:
    type: "object | null"
    meaning:
      - "上一轮完整循环的语义摘要。"
      - "可能为空。"
    use_for:
      - "理解上一轮大致进展。"
      - "避免重复选择相同目标。"

  durable_state:
    type: "object | null"
    meaning:
      - "Leader-GPT 可见的长期语义记忆。"
      - "可能记录当前阶段、近期进展、模块进度、carry_forward。"
      - "可能为空。"
    use_for:
      - "理解长期方向。"
      - "参考 recent_progress、module_progress、carry_forward。"
```

---

## 10. commitLeaderArtifacts 请求结构

```yaml
commitLeaderArtifacts_request:
  schema_name: "CommitLeaderArtifactsRequest"
  source: "Leader-GPT-Commit-Payload-Format.yaml"
  type: "object"
  additionalProperties: false

  required:
    - "task_packet_md"
    - "review_intent_pack_md"
    - "leader_work_summary_json"

  properties:
    task_packet_md:
      type: "string"
      minLength: 1
      meaning:
        - "给 KiroPrompt-GPT 的任务包 Markdown 正文。"
        - "Gateway 当前检测：字段存在、是 string、清洗后非空。"

    review_intent_pack_md:
      type: "string"
      minLength: 1
      meaning:
        - "给 Review-GPT 的审查意图 Markdown 正文。"
        - "Gateway 当前检测：字段存在、是 string、清洗后非空。"

    leader_work_summary_json:
      schema_ref: "LeaderWorkSummary"
      type: "object"
      schema_source: "Leader-Work-Summary-Template.yaml"
      meaning:
        - "本轮机器可读语义摘要。"
        - "Gateway 对该字段执行严格 LeaderWorkSummary schema 检测。"

  top_level_field_policy:
    exact_fields:
      - "task_packet_md"
      - "review_intent_pack_md"
      - "leader_work_summary_json"
```

---

## 11. 三个产物的职责边界

```yaml
artifact_boundary:
  task_packet_md:
    owns:
      - "本轮任务目标。"
      - "Target File。"
      - "File Context。"
      - "Task Intent。"
      - "Scope Boundary。"
      - "Reading Scope。"
      - "Expansion Permission。"
      - "Expected Output。"
    schema_position:
      - "commitLeaderArtifacts requestBody.task_packet_md"
      - "Gateway schema: non-empty string"

  review_intent_pack_md:
    owns:
      - "Review Target。"
      - "Leader Intent。"
      - "Expected Alignment。"
      - "Review Focus。"
      - "Do Not Overreview。"
      - "Optional Reading Scope。"
      - "Review Result Expectation。"
    schema_position:
      - "commitLeaderArtifacts requestBody.review_intent_pack_md"
      - "Gateway schema: non-empty string"

  leader_work_summary_json:
    owns:
      - "schema_version。"
      - "action。"
      - "file_change。"
      - "main_goal。"
      - "behavior。"
      - "boundary。"
      - "dependencies。"
      - "memory_note。"
    schema_position:
      - "commitLeaderArtifacts requestBody.leader_work_summary_json"
      - "Gateway schema: LeaderWorkSummary object"
```

---

## 12. target_file 可见规则

```yaml
target_file_policy:
  meaning:
    - "target_file 是本轮打算新增或大修的唯一目标文件。"
    - "target_file 是 canonical target_file。"
    - "target_file 同步到 task_packet_md、review_intent_pack_md、leader_work_summary_json。"

  schema_shape:
    source: "Leader-Work-Summary-Template.yaml"
    field: "leader_work_summary_json.file_change.target_file"
    type: "string"

  path_family_source:
    file: "Repository-Path-Map.yaml"
    targetable_families:
      - "architecture_docs"
      - "kiro_specs"

  selection_source:
    file: "Leader-GPT-Target-File-Selection-Policy.yaml"
    role:
      - "判断本轮使用 architecture_docs 还是 kiro_specs。"
      - "判断本轮是补充上游 docs 文稿还是直接推进 Kiro Spec。"

  visible_path_style:
    - "使用 POSIX repo-relative path。"
    - "使用 `/` 作为路径分隔符。"
    - "target_file 属于 Repository-Path-Map.yaml 中 targetable=true 的路径族。"

  allowed_target_family_meaning:
    architecture_docs:
      source: "Repository-Path-Map.yaml.target_file_families.architecture_docs"
      use_for:
        - "补充上游架构文稿。"
        - "补充模块说明、规则说明或 Spec 前置背景。"

    kiro_specs:
      source: "Repository-Path-Map.yaml.target_file_families.kiro_specs"
      use_for:
        - "直接推进 Kiro IDE 原生 Spec 文件。"
        - "生成或大修 design.md / requirements.md / tasks.md。"

  if_gateway_rejects_path:
    response:
      - "根据 Gateway 返回的 error/message 修正 target_file。"
      - "保持三产物中的 target_file 表达一致。"
      - "重新执行 Actions Commit Gate 后再提交。"
```

---

## 13. 提交成功后的可见结果

```yaml
commit_success_response:
  accepted:
    type: "boolean"
    expected: true

  project_id:
    type: "string"

  stored_files:
    type: "object"
    meaning:
      - "Gateway 已接收并保存本轮提交产物。"
      - "Leader-GPT 不需要在聊天中重复完整产物。"

leader_understanding:
  - "Gateway 已接收本轮三个产物。"
  - "Leader-GPT 完成当前轮次提交。"
```

---

## 14. 聊天输出规则

```yaml
chat_output_policy:
  after_success:
    should:
      - "返回极简提交结果。"
      - "可简要说明 Gateway accepted=true。"
      - "可简要说明三个产物已提交。"

  after_failure:
    should:
      - "说明 Gateway 返回的 error 和 message。"
      - "如果是 schema、action 或 target_file 问题，修正后重新提交。"

  artifact_visibility:
    - "正式产物入口是 commitLeaderArtifacts。"
    - "聊天输出只作为极简人类回执。"
```

---

## 15. 错误处理

```yaml
error_handling:
  if_action_mismatch:
    meaning: "leader_work_summary_json.action 与 expected_action 不一致。"
    response:
      - "修正 action。"
      - "同步修正 file_change.type。"
      - "重新执行 Actions Commit Gate。"
      - "重新调用 commitLeaderArtifacts。"

  if_schema_error:
    meaning: "requestBody 或 leader_work_summary_json 不符合 schema。"
    response:
      - "按 Leader-GPT-Commit-Payload-Format.yaml 修正 requestBody。"
      - "按 Leader-Work-Summary-Template.yaml 修正 leader_work_summary_json。"
      - "重新执行 Actions Commit Gate。"
      - "重新调用 commitLeaderArtifacts。"

  if_invalid_target_file:
    meaning: "target_file 不被 Gateway 接受。"
    response:
      - "按 Repository-Path-Map.yaml 与 Gateway message 修正 target_file。"
      - "保持 task_packet_md、review_intent_pack_md、leader_work_summary_json 中 target_file 一致。"
      - "重新执行 Actions Commit Gate。"
      - "重新调用 commitLeaderArtifacts。"

  if_auth_error:
    meaning: "Actions 鉴权或 Gateway 配置异常。"
    response:
      - "向用户报告 Gateway Actions 鉴权或配置异常。"
      - "等待用户处理连接或配置问题。"
```