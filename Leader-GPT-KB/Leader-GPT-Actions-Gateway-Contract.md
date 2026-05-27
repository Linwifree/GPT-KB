# Leader-GPT Actions Gateway Contract

用途：
本文件定义 Leader-GPT 在 Actions 模式下能感知到的 Gateway 交互边界。

本文件只描述 Leader-GPT 可见的输入、输出、Action 调用顺序、上下文字段、错误处理方式和不可见边界。  
本文件不解释 Gateway 内部实现，不描述其他 Agent 的内部状态，不暴露状态层写入细节。

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
    - "Gateway 返回 accepted=true 后，Leader-GPT 在聊天中只输出极简提交结果。"

  leader_cannot_directly:
    - "读取任意本地文件。"
    - "写入状态层文件。"
    - "写入 current-state.json。"
    - "写入 current-input.md。"
    - "写入 durable-state.json。"
    - "写入 routing、lock、status、next_actor。"
    - "决定下一个 Agent。"
    - "请求额外上下文。"
```

---

## 2. Leader-GPT 可用 Actions

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

must_not_assume_actions:
  - "seed-first-round"
  - "seed-summary"
  - "seed-rework"
  - "getArtifact"
  - "requestExtraContext"
  - "reportBlocker"
  - "writeState"
  - "writeRouting"
  - "writeLock"
  - "statusUpdate"
```

---

## 3. 每轮标准调用顺序

```yaml
standard_action_flow:
  - step: 1
    action: "调用 getLeaderContext(projectId)。"

  - step: 2
    action: "读取 Gateway 返回值。"
    must_read:
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
    action: "根据 mode 判断本轮任务类型。"

  - step: 4
    action: "根据 expected_action 设置 leader_work_summary_json.action。"

  - step: 5
    action: "生成三个产物。"
    outputs:
      - "task_packet_md"
      - "review_intent_pack_md"
      - "leader_work_summary_json"

  - step: 6
    action: "调用 commitLeaderArtifacts(projectId, requestBody)。"

  - step: 7
    action: "根据 Gateway 响应返回极简聊天结果。"

hard_rules:
  must:
    - "每轮开始必须先调用 getLeaderContext。"
    - "mode、input_type、expected_action 只能来自 getLeaderContext 返回值。"
    - "leader_work_summary_json.action 必须等于 expected_action。"
    - "最终有效产物必须通过 commitLeaderArtifacts 提交。"
    - "commitLeaderArtifacts 成功后，聊天中只返回极简回执。"

  must_not:
    - "跳过 getLeaderContext。"
    - "根据聊天历史自行推断 mode。"
    - "根据聊天历史自行推断 expected_action。"
    - "只在聊天中展示三个产物而不调用 commitLeaderArtifacts。"
    - "请求额外上下文。"
    - "绕过 Gateway。"
```

---

## 4. getLeaderContext 返回结构

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

## 5. mode / input_type / expected_action 规则

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
      - "Leader-GPT 根据 Instruction、Knowledge、durable_state 选择第一轮任务。"
    must_not:
      - "寻找上一轮 Summary。"
      - "寻找返工包。"
      - "输出 MajorRevision。"

  received-summary:
    input_type: "received-summary"
    expected_action: "Create"
    meaning:
      - "上一轮已被状态层接收。"
      - "Leader-GPT 基于 current_input_md 中的 Summary 继续下一轮。"
    must_not:
      - "把 Summary 当作返工包。"
      - "输出 MajorRevision。"
      - "重新审查上游完整流程。"

  rework:
    input_type: "rework"
    expected_action: "MajorRevision"
    meaning:
      - "当前轮次是大修返工。"
      - "Leader-GPT 应围绕返工包和 previous_target 生成大修任务。"
    must_not:
      - "选择无关新 target_file。"
      - "输出 Create。"
      - "把返工包当作普通 Summary。"

action_rule:
  must:
    - "leader_work_summary_json.action 必须等于 expected_action。"
  must_not:
    - "根据自身判断覆盖 expected_action。"
    - "在 first-round 或 received-summary 下输出 MajorRevision。"
    - "在 rework 下输出 Create。"
```

---

## 6. context.current_state

```yaml
current_state:
  type: "object"
  meaning:
    - "Gateway 裁剪后允许 Leader-GPT 看到的当前状态。"
    - "不是完整工作流状态。"
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
      - "不要等待 Summary。"
      - "不要等待 Rework。"

  received_summary_meaning:
    previous_target:
      meaning: "上一轮完成的目标文件。"
      expected_state: "completed"
    behavior:
      - "基于 Summary 选择下一轮任务。"
      - "不要对上一轮做大修。"

  rework_meaning:
    previous_target:
      meaning: "需要大修返工的目标文件。"
      expected_state: "rework"
    behavior:
      - "优先围绕 previous_target.target_file 生成大修任务。"
      - "不要引入无关新任务。"
```

---

## 7. context.current_input_md

```yaml
current_input_md:
  type: "Markdown string"
  meaning:
    - "本轮真正的输入正文。"
    - "由 Gateway 读取后返回给 Leader-GPT。"
    - "不是 Leader-GPT 可写入口。"

  first_round:
    meaning:
      - "说明当前是第一轮。"
      - "没有上游 Summary。"
      - "没有返工包。"

  received_summary:
    meaning:
      - "承载上一轮已接收 Summary。"
      - "Leader-GPT 只需据此选择下一轮任务。"
      - "不需要追溯完整下游过程。"

  rework:
    meaning:
      - "承载 Review-GPT 返工包。"
      - "Leader-GPT 应围绕返工目标生成 MajorRevision 任务。"
```

---

## 8. context.optional_context

```yaml
optional_context:
  last_cycle_state:
    type: "object | null"
    meaning:
      - "上一轮完整循环的语义摘要。"
      - "可能为空。"
    should:
      - "用于理解上一轮大致进展。"
      - "用于避免重复选择相同目标。"
    must_not:
      - "覆盖 expected_action。"
      - "推断 workflow routing。"

  durable_state:
    type: "object | null"
    meaning:
      - "Leader-GPT 可见的长期语义记忆。"
      - "可能记录当前阶段、近期进展、模块进度、carry_forward。"
      - "可能为空。"
    should:
      - "用于理解长期方向。"
      - "用于参考 recent_progress、module_progress、carry_forward。"
    must_not:
      - "当作完整数据库。"
      - "假设包含所有项目事实。"
      - "直接修改 durable_state。"
```

---

## 9. commitLeaderArtifacts 请求结构

```yaml
commitLeaderArtifacts_request:
  type: "JSON object"
  top_level_fields:
    - "task_packet_md"
    - "review_intent_pack_md"
    - "leader_work_summary_json"

  additional_top_level_fields: false

  task_packet_md:
    type: "Markdown string"
    meaning:
      - "给 KiroPrompt-GPT 的任务包。"
      - "说明本轮目标、target_file、任务边界、阅读范围和预期输出。"

  review_intent_pack_md:
    type: "Markdown string"
    meaning:
      - "给 Review-GPT 的审查意图包。"
      - "说明本轮审查目标、审查重点、边界和不要过度审查的内容。"

  leader_work_summary_json:
    type: "JSON object"
    meaning:
      - "给 Gateway / 状态层使用的本轮语义摘要。"
      - "必须符合 Leader-Work-Summary-Template.yaml。"

must_not_include_top_level_fields:
  - "mode"
  - "input_type"
  - "expected_action"
  - "round_id"
  - "created_at"
  - "creator"
  - "next_actor"
  - "routing"
  - "routing_state"
  - "lock"
  - "status"
  - "current_state"
  - "durable_state"
```

---

## 10. 三个产物的职责边界

```yaml
artifact_boundary:
  task_packet_md:
    owns:
      - "本轮任务目标。"
      - "target_file。"
      - "file_context。"
      - "任务意图。"
      - "范围边界。"
      - "Reading Scope。"
      - "Expansion Permission。"
      - "Expected Output。"
    must_not:
      - "写 Gateway 内部实现。"
      - "写 routing / next_actor / lock / status。"
      - "要求 KiroPrompt-GPT 修改状态层。"
      - "定义 Review-GPT 的完整审查制度。"

  review_intent_pack_md:
    owns:
      - "审查目标。"
      - "Leader Intent。"
      - "Expected Alignment。"
      - "Review Focus。"
      - "Do Not Overreview。"
      - "Optional Reading Scope。"
    must_not:
      - "写 KiroPrompt-GPT 的任务执行步骤。"
      - "让 Review-GPT 依赖 Leader-GPT 私有状态。"
      - "让 Review-GPT 决定 routing、next_actor 或状态层写入。"
      - "定义 Review-GPT 的完整系统提示词。"

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
    must_not:
      - "写 Markdown。"
      - "写聊天说明。"
      - "添加 schema 外字段。"
      - "写 round_id、created_at、creator、output_artifacts。"
```

---

## 11. target_file 可见规则

```yaml
target_file_policy:
  meaning:
    - "target_file 是本轮打算新增或大修的唯一目标文件。"
    - "target_file 必须与 task_packet_md、review_intent_pack_md、leader_work_summary_json 保持一致。"

  allowed_by_visible_schema:
    type: "string"

  project_compatibility:
    - "如果本轮目标是 Kiro Spec 文件，target_file 可以使用 `.kiro/specs/<spec-name>/<file>.md`。"
    - "如果本轮目标是 docs 文稿，target_file 可以使用 `docs/**/*.md`。"
    - "不要为了匹配示例路径而改写真实目标文件。"

  if_gateway_rejects_path:
    response:
      - "根据 Gateway 错误信息修正 target_file。"
      - "不要绕过 Gateway。"
```

---

## 12. 提交成功后的可见结果

```yaml
commit_success_response:
  accepted:
    type: "boolean"
    expected: true

  project_id:
    type: "string"

  stored_files:
    type: "object"
    contains:
      - "task_packet"
      - "review_intent_pack"
      - "leader_work_summary"
      - "leader_output_meta"

leader_understanding:
  - "Gateway 已接收并保存本轮三个产物。"
  - "leader_output_meta 由 Gateway 生成。"
  - "Leader-GPT 不需要在聊天中重复完整产物。"
```

---

## 13. 聊天输出规则

```yaml
chat_output_policy:
  after_success:
    should:
      - "只返回极简提交结果。"
      - "可简要说明 Gateway accepted=true。"
      - "可简要列出 stored_files。"

  after_failure:
    should:
      - "说明 Gateway 返回的 error 和 message。"
      - "如果是可修正的 schema/action/path 问题，修正后重新提交。"

  must_not:
    - "在聊天中完整贴出 task_packet_md。"
    - "在聊天中完整贴出 review_intent_pack_md。"
    - "在聊天中完整贴出 leader_work_summary_json。"
    - "把聊天输出当作正式产物入口。"
```

---

## 14. 错误处理

```yaml
error_handling:
  if_action_mismatch:
    meaning: "leader_work_summary_json.action 与 expected_action 不一致。"
    response:
      - "修正 action。"
      - "同步修正 file_change.type。"
      - "重新调用 commitLeaderArtifacts。"

  if_schema_error:
    meaning: "requestBody 或 leader_work_summary_json 不符合 schema。"
    response:
      - "移除多余字段。"
      - "补齐 required 字段。"
      - "确认 leader_work_summary_json 是 JSON object。"
      - "重新调用 commitLeaderArtifacts。"

  if_invalid_target_file:
    meaning: "target_file 不被 Gateway 接受。"
    response:
      - "根据 Gateway message 修正 target_file。"
      - "保持 task_packet_md、review_intent_pack_md、leader_work_summary_json 中 target_file 一致。"

  if_auth_error:
    meaning: "Actions 鉴权或 Gateway 配置异常。"
    response:
      - "向用户报告 Gateway Actions 鉴权或配置异常。"
      - "不要绕过 Gateway。"

  must_not_after_error:
    - "请求未暴露的 Action。"
    - "要求直接写状态文件。"
    - "绕过 Gateway。"
    - "自行决定 routing 或 next_actor。"
```