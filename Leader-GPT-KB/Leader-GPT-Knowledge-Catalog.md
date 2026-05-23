# Leader-GPT Knowledge Catalog

```yaml
knowledge_catalog:
  id: "leader-gpt-knowledge-catalog"
  title: "Leader-GPT Knowledge Catalog"
  version: "2.0.0"

  purpose:
    - "说明 Leader-GPT 可用知识文件的用途、调用时机与边界。"
    - "帮助 Leader-GPT 按信息节点选择正确文件，而不是把所有知识文件平均使用。"
    - "区分 Actions/Gateway 类、项目背景类、Specs 方向类、阅读协议类、产物模板类、提交闸门类与 legacy 兼容类文件。"

  instruction_offload_contract:
    meaning:
      - "Leader-GPT Instruction 只保留角色启动、Actions 主工作流、极简导航和硬边界。"
      - "本 Catalog 承接详细知识文件导航、信息节点路由、默认调用顺序和模板细节归属说明。"
    catalog_owns:
      - "每个 Knowledge 文件的详细用途。"
      - "每个信息节点应调用哪些文件。"
      - "first-round / received-summary / rework 的处理参考。"
      - "Actions commit 产物的模板归属边界。"
      - "legacy marker 输出文件的兼容定位。"

  usage_principle:
    - "先判断当前阶段、输入模式和任务类型，再选择知识文件。"
    - "Actions/Gateway 类文件用于理解 Leader-GPT 可见输入输出边界。"
    - "项目背景类文件用于理解模块与边界。"
    - "模板类文件用于生成 commitLeaderArtifacts 的字段内容。"
    - "提交闸门类文件只在 commitLeaderArtifacts 前使用。"
    - "不得把模板文件当作项目背景。"
    - "不得把项目背景文件当作输出格式模板。"
    - "不得把 legacy envelope / release gate 当作 Actions 主链路。"
```

---

## 1. Actions / Gateway 类

```yaml
actions_gateway_files:
  - id: "leader-gpt-actions-gateway-contract"
    file: "Leader-GPT-Actions-Gateway-Contract.md"
    type: "actions_gateway_contract"
    priority: "highest"
    use_when:
      - "理解 Leader-GPT 如何通过 Actions 与 Gateway 交互。"
      - "确认 getLeaderContext / commitLeaderArtifacts 的职责。"
      - "确认 mode / input_type / expected_action 的意义。"
      - "确认 commitLeaderArtifacts requestBody 结构。"
      - "确认 Leader-GPT 看得到什么、看不到什么、不能直接写什么。"
    do_not_use_for:
      - "替代项目背景。"
      - "替代 Kiro Specs 定义。"
      - "替代 task_packet_md 正文模板。"
      - "替代 review_intent_pack_md 正文模板。"

  - id: "leader-gpt-actions-commit-gate"
    file: "Leader-GPT-Actions-Commit-Gate.yaml"
    type: "actions_commit_gate"
    priority: "highest_before_commit"
    use_when:
      - "调用 commitLeaderArtifacts 前进行自检。"
      - "确认已调用 getLeaderContext。"
      - "确认 expected_action 来自 Gateway。"
      - "确认 requestBody 只有三个顶层字段。"
      - "确认 leader_work_summary_json 是 JSON object。"
      - "确认三个产物围绕同一个 target_file。"
    do_not_use_for:
      - "选择 target_file。"
      - "生成项目背景。"
      - "生成 task_packet_md 正文。"
      - "生成 review_intent_pack_md 正文。"
      - "解释 Gateway 内部实现。"
```

---

## 2. 当前方向类

```yaml
direction_files:
  - id: "current-specs-completion-direction"
    file: "current-specs-completion-direction.yaml"
    type: "current_stage_direction"
    priority: "highest_project_context"
    use_when:
      - "判断当前项目阶段。"
      - "判断当前任务是否仍属于 Specs completion。"
      - "判断 Specs 是否应推进到 design.md / requirements.md / tasks.md。"
      - "判断 Specs 是否应保持彼此协调，而不是孤立点。"
      - "判断低风险底层框架 Spec 与后续 Specs 的关系。"
    do_not_use_for:
      - "替代 Gateway Actions 合约。"
      - "替代具体模块背景。"
      - "替代 task_packet_md 模板。"
      - "替代 review_intent_pack_md 模板。"
```

---

## 3. 项目知识类

```yaml
project_knowledge_files:
  - id: "KB-module-knowledge-graph"
    file: "KB-module-knowledge-graph.yaml"
    type: "module_relation_graph"
    priority: "high"
    use_when:
      - "判断模块之间的关系。"
      - "判断 Spec 是否与其他模块边界冲突。"
      - "判断 Stable Core / Variable Layer / Extension Layer 关系。"
      - "判断 Auth / Dashboard / Admin / Marketing / Extension Slot 的职责边界。"
      - "为 task_packet_md 和 review_intent_pack_md 提供模块边界依据。"
    do_not_use_for:
      - "替代 Actions Gateway 合约。"
      - "替代 KiroPrompt-GPT Task Packet 模板。"
      - "替代 Leader Work Summary 模板。"

  - id: "KB-source-distillation"
    file: "KB-source-distillation.yaml"
    type: "source_distillation"
    priority: "high"
    use_when:
      - "快速理解已纳入架构文稿的主要内容。"
      - "提取目标模块的职责、边界、风险和非目标。"
      - "为 Task Intent 和 Review Intent 提供压缩背景。"
      - "辅助判断源文件用途和 Reading Scope 中的文件角色。"
      - "减少对原始长文稿的依赖。"
    do_not_use_for:
      - "替代模块知识图谱。"
      - "替代 Gateway context。"
      - "替代 reading-layer-quantization-standard.yaml。"
```

---

## 4. Kiro Specs 定义类

```yaml
specs_definition_files:
  - id: "kiro-specs-definition"
    file: "kiro-specs-definition.yaml"
    type: "kiro_specs_definition"
    priority: "high"
    use_when:
      - "需要理解 Kiro Specs 的 design.md / requirements.md / tasks.md 三层结构。"
      - "需要判断本轮 target_file 属于哪个 Spec 层。"
      - "需要写 task_packet_md 的 Target File 与 File Context。"
      - "需要写 review_intent_pack_md 的 Related Spec / File Context。"
      - "需要说明本轮期望创建、推进或大修哪个 Spec 文件。"
    do_not_use_for:
      - "替代项目模块背景。"
      - "替代 Actions Gateway 合约。"
      - "替代 task_packet_md 模板。"
```

---

## 5. 阅读协议类

```yaml
reading_protocol_files:
  - id: "reading-layer-quantization-standard"
    file: "reading-layer-quantization-standard.yaml"
    type: "reading_protocol"
    priority: "high"
    use_when:
      - "为 task_packet_md 编写 Reading Scope。"
      - "为 review_intent_pack_md 编写 Optional Reading Scope。"
      - "为指定文件标注 R? + D?。"
      - "设置扩读条件。"
      - "设置停止扩读条件。"
      - "避免使用“相关文件”“按需阅读”“看情况补读”等模糊表达。"
    do_not_use_for:
      - "解释项目架构。"
      - "生成 Review-GPT 的审查结论。"
      - "替代 KiroPrompt-GPT 自身的阅读执行规则。"
      - "替代 Gateway context。"
```

---

## 6. Actions 产物模板类

```yaml
leader_artifact_templates:
  - id: "kiro-prompt-task-packet-template"
    file: "KiroPrompt-GPT-Task-Packet-Template.md"
    type: "markdown_artifact_template"
    action_field: "task_packet_md"
    stored_by_gateway_as: "task-packet.md"
    future_consumer: "KiroPrompt-GPT"
    use_when:
      - "生成 commitLeaderArtifacts.task_packet_md。"
      - "说明本轮 target_file、file_context、任务意图、范围边界、Reading Scope、扩读条件和预期输出。"
      - "为 KiroPrompt-GPT 提供生成后续执行提示的任务依据。"
    do_not_use_for:
      - "生成 review_intent_pack_md。"
      - "生成 leader_work_summary_json。"
      - "记录 routing、next_actor、lock 或 status。"
      - "要求 KiroPrompt-GPT 修改状态层。"

  - id: "review-intent-packet-template"
    file: "Review-Intent-Packet-Template.md"
    type: "markdown_artifact_template"
    action_field: "review_intent_pack_md"
    stored_by_gateway_as: "review-intent-pack.md"
    future_consumer: "Review-GPT"
    use_when:
      - "生成 commitLeaderArtifacts.review_intent_pack_md。"
      - "说明本轮 Review Target、Leader Intent、Expected Alignment、Review Focus 和 Do Not Overreview。"
      - "为 Review-GPT 提供审查意图。"
    do_not_use_for:
      - "生成 task_packet_md。"
      - "替代 Review-GPT 自身审查规则。"
      - "生成 leader_work_summary_json。"
      - "决定 routing、next_actor、lock 或 status。"

  - id: "leader-work-summary-template"
    file: "Leader-Work-Summary-Template.yaml"
    type: "json_artifact_template"
    action_field: "leader_work_summary_json"
    stored_by_gateway_as: "leader-work-summary.json"
    use_when:
      - "生成 commitLeaderArtifacts.leader_work_summary_json。"
      - "记录 schema_version、action、file_change、main_goal、behavior、boundary、dependencies、memory_note。"
      - "确认 action 等于 getLeaderContext.expected_action。"
      - "确认 file_change.type 与 action 配对。"
    do_not_use_for:
      - "生成 Markdown 任务包。"
      - "生成 Markdown 审查意图包。"
      - "生成 round_id、created_at、creator 或 output_artifacts。"
      - "写 routing、next_actor、lock 或 status。"
```

---

## 7. Legacy 兼容类

```yaml
legacy_compatibility_files:
  - id: "round-meta-template"
    file: "Round-Meta-Template.md"
    type: "legacy_json_output_template"
    status: "legacy_chat_output_compatibility"
    use_when:
      - "仅在非 Actions 模式、人工调试或用户明确要求旧三块聊天输出时参考。"
    do_not_use_for:
      - "Actions 主链路。"
      - "commitLeaderArtifacts.leader_work_summary_json。"
      - "生成 round_id、created_at、creator 或 output_artifacts。"

  - id: "leader-round-output-envelope-standard"
    file: "Leader-GPT-Round-Output-Envelope-Standard.md"
    type: "legacy_output_envelope_standard"
    status: "legacy_chat_output_compatibility"
    use_when:
      - "仅在非 Actions 模式、人工调试或用户明确要求聊天中展示完整 marker 产物时参考。"
    do_not_use_for:
      - "Actions 主链路。"
      - "commitLeaderArtifacts requestBody。"
      - "判断项目方向。"
      - "替代 Actions Commit Gate。"

  - id: "leader-output-release-gate"
    file: "Leader-GPT-Output-Release-Gate.yaml"
    type: "legacy_marker_release_gate"
    status: "legacy_chat_output_compatibility"
    use_when:
      - "仅在非 Actions 模式或人工调试旧 marker 输出时参考。"
    do_not_use_for:
      - "Actions 主链路。"
      - "commitLeaderArtifacts 前的正式自检。"
      - "替代 Leader-GPT-Actions-Commit-Gate.yaml。"
```

---

## 8. 信息节点路由

```yaml
information_node_routing:
  get_leader_context:
    purpose: "调用 getLeaderContext 读取本轮允许上下文。"
    use:
      - "leader-gpt-actions-gateway-contract"

  interpret_gateway_context:
    purpose: "解释 mode、input_type、expected_action、current_state、current_input_md、optional_context。"
    use:
      - "leader-gpt-actions-gateway-contract"
      - "current-specs-completion-direction"
      - "KB-source-distillation"
      - "KB-module-knowledge-graph"

  decide_mode_behavior:
    purpose: "根据 mode 与 expected_action 判断本轮是 first-round、received-summary 还是 rework。"
    use:
      - "leader-gpt-actions-gateway-contract"

  choose_target_spec_or_target_file:
    purpose: "判断本轮应新增或大修哪个唯一 target_file。"
    use:
      - "current-specs-completion-direction"
      - "KB-module-knowledge-graph"
      - "KB-source-distillation"
      - "kiro-specs-definition"
      - "leader-gpt-actions-gateway-contract"

  understand_project_background:
    purpose: "理解项目背景、模块关系和当前目标 Spec 所处位置。"
    use:
      - "KB-module-knowledge-graph"
      - "KB-source-distillation"
      - "kiro-specs-definition"

  build_task_packet_md:
    purpose: "生成 commitLeaderArtifacts.task_packet_md。"
    use:
      - "kiro-prompt-task-packet-template"
      - "reading-layer-quantization-standard"
      - "kiro-specs-definition"
      - "current-specs-completion-direction"
      - "KB-source-distillation"
      - "KB-module-knowledge-graph"

  build_review_intent_pack_md:
    purpose: "生成 commitLeaderArtifacts.review_intent_pack_md。"
    use:
      - "review-intent-packet-template"
      - "reading-layer-quantization-standard"
      - "current-specs-completion-direction"
      - "KB-module-knowledge-graph"
      - "KB-source-distillation"
      - "kiro-specs-definition"

  build_leader_work_summary_json:
    purpose: "生成 commitLeaderArtifacts.leader_work_summary_json。"
    use:
      - "leader-work-summary-template"
      - "leader-gpt-actions-gateway-contract"

  validate_before_commit:
    purpose: "调用 commitLeaderArtifacts 前进行 Actions Commit Gate 自检。"
    use:
      - "leader-gpt-actions-commit-gate"

  commit_leader_artifacts:
    purpose: "通过 commitLeaderArtifacts 一次性提交三个产物。"
    use:
      - "leader-gpt-actions-gateway-contract"

  handle_gateway_response:
    purpose: "根据 Gateway accepted / error 返回极简聊天结果。"
    use:
      - "leader-gpt-actions-gateway-contract"
      - "leader-gpt-actions-commit-gate"
```

---

## 9. 默认调用顺序

```yaml
default_execution_order:
  - step: 1
    node: "get_leader_context"
    goal: "调用 getLeaderContext(projectId)，读取本轮允许上下文。"

  - step: 2
    node: "interpret_gateway_context"
    goal: "读取 mode、input_type、expected_action、current_state、current_input_md、optional_context。"

  - step: 3
    node: "decide_mode_behavior"
    goal: "根据 mode 判断 first-round / received-summary / rework 的行为。"

  - step: 4
    node: "choose_target_spec_or_target_file"
    goal: "确定本轮唯一 target_file；rework 模式优先使用 previous_target.target_file。"

  - step: 5
    node: "understand_project_background"
    goal: "只读取与本轮目标相关的项目背景、Spec 定义和模块边界。"

  - step: 6
    node: "build_task_packet_md"
    goal: "生成给 KiroPrompt-GPT 的 Markdown string。"

  - step: 7
    node: "build_review_intent_pack_md"
    goal: "生成给 Review-GPT 的 Markdown string。"

  - step: 8
    node: "build_leader_work_summary_json"
    goal: "生成符合 Leader-Work-Summary-Template.yaml 的 JSON object。"

  - step: 9
    node: "validate_before_commit"
    goal: "检查 requestBody、action、file_change.type、target_file 和职责边界。"

  - step: 10
    node: "commit_leader_artifacts"
    goal: "调用 commitLeaderArtifacts 提交 task_packet_md、review_intent_pack_md、leader_work_summary_json。"

  - step: 11
    node: "handle_gateway_response"
    goal: "成功时返回极简回执；失败时报告 Gateway error / message。"
```

---

## 10. 模板细节归属

```yaml
template_detail_policy:
  actions_main_path:
    - "task_packet_md 细节以 KiroPrompt-GPT-Task-Packet-Template.md 为准。"
    - "review_intent_pack_md 细节以 Review-Intent-Packet-Template.md 为准。"
    - "leader_work_summary_json 字段以 Leader-Work-Summary-Template.yaml 为准。"
    - "Actions 交互边界以 Leader-GPT-Actions-Gateway-Contract.md 为准。"
    - "提交前自检以 Leader-GPT-Actions-Commit-Gate.yaml 为准。"

  legacy_path:
    - "Round-Meta-Template.md 仅作为旧聊天三块输出兼容文件。"
    - "Leader-GPT-Round-Output-Envelope-Standard.md 仅作为旧 marker envelope 兼容文件。"
    - "Leader-GPT-Output-Release-Gate.yaml 仅作为旧 marker release gate 兼容文件。"

  rule:
    - "Instruction 不重复模板细节。"
    - "Catalog 只做导航和职责分配，不替代模板正文。"
```

---

## 11. 使用边界

```yaml
catalog_boundary:
  must:
    - "把本目录作为知识文件导航，而不是任务输出模板。"
    - "按信息节点调用文件，不把所有文件平均使用。"
    - "优先使用 Actions/Gateway 文件确认 Leader-GPT 可见输入输出边界。"
    - "优先使用方向文件确认当前项目阶段。"
    - "优先使用模板文件生成对应 Actions 产物。"
    - "调用 commitLeaderArtifacts 前必须使用 Actions Commit Gate 自检。"

  must_not:
    - "把目录文件内容写进最终 task_packet_md 正文。"
    - "把模板说明当作项目背景。"
    - "把项目背景文件当作 Actions requestBody 约束。"
    - "把 legacy envelope 文件当作 Actions 主链路。"
    - "把 legacy release gate 文件当作 Actions Commit Gate。"
    - "跳过 getLeaderContext。"
    - "跳过 leader_work_summary_json。"
    - "跳过 Actions Commit Gate。"
    - "跳过 commitLeaderArtifacts。"
```
