# Leader-GPT Knowledge Catalog

```yaml
knowledge_catalog:
  id: "leader-gpt-knowledge-catalog"
  title: "Leader-GPT Knowledge Catalog"
  version: "1.0.0"

  purpose:
    - "说明 Leader-GPT 可用知识文件的用途、调用时机与边界。"
    - "帮助 Leader-GPT 按信息节点选择正确文件，而不是把所有知识文件平均使用。"
    - "区分项目背景类、Specs 方向类、阅读协议类、输出模板类、输出 envelope 类与发布闸门类文件。"

  usage_principle:
    - "先判断当前阶段与任务类型，再选择知识文件。"
    - "项目背景类文件用于理解模块与边界。"
    - "模板类文件用于生成固定输出块。"
    - "输出 envelope 类文件用于组合三个输出块。"
    - "发布闸门类文件只在最终输出前使用。"
    - "不得把模板文件当作项目背景。"
    - "不得把项目背景文件当作输出格式模板。"
    - "不得把 envelope 文件当作 release gate。"
    - "不得把 release gate 文件当作输出模板。"
````

---

## 1. 当前方向类

```yaml
direction_files:
  - id: "current-specs-completion-direction"
    file: "current-specs-completion-direction.yaml"
    type: "current_stage_direction"
    priority: "highest"
    use_when:
      - "判断当前项目阶段。"
      - "判断当前任务是否仍属于 Specs completion。"
      - "判断 Specs 是否应推进到 design.md / requirements.md / tasks.md。"
      - "判断 Specs 是否应保持彼此协调，而不是孤立点。"
      - "判断低风险底层框架 Spec 与后续 Specs 的关系。"
    do_not_use_for:
      - "替代具体模块背景。"
      - "替代任务包模板。"
      - "替代 Review-GPT 审查意图模板。"
```

---

## 2. 项目知识类

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
      - "为 Review Intent Packet 提供审查对齐依据。"
    do_not_use_for:
      - "替代 KiroPrompt-GPT Task Packet 模板。"
      - "替代 Round Meta 模板。"

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
      - "替代任务包中的 GitHub 文件阅读范围。"
      - "替代 reading-layer-quantization-standard.yaml。"
```

---

## 3. Kiro Specs 定义类

```yaml
specs_definition_files:
  - id: "kiro-specs-definition"
    file: "kiro-specs-definition.yaml"
    type: "kiro_specs_definition"
    priority: "high"
    use_when:
      - "需要理解 Kiro Specs 的 design.md / requirements.md / tasks.md 三层结构。"
      - "需要判断本轮 target_file 属于哪个 Spec 层。"
      - "需要写 KiroPrompt-GPT Task Packet 的 Target File 与 File Context。"
      - "需要说明本轮期望创建或补全哪个 Spec 文件。"
    do_not_use_for:
      - "替代项目模块背景。"
      - "替代 KiroPrompt-GPT Task Packet 模板。"
```

---

## 4. 阅读协议类

```yaml
reading_protocol_files:
  - id: "reading-layer-quantization-standard"
    file: "reading-layer-quantization-standard.yaml"
    type: "reading_protocol"
    priority: "high"
    use_when:
      - "为 KiroPrompt-GPT Task Packet 编写 Reading Scope。"
      - "为指定 GitHub 文件标注 R? + D?。"
      - "设置扩读条件。"
      - "设置停止扩读条件。"
      - "避免使用“相关文件”“按需阅读”“看情况补读”等模糊表达。"
    do_not_use_for:
      - "解释项目架构。"
      - "生成 Review-GPT 的审查结论。"
      - "替代 KiroPrompt-GPT 自身的阅读执行规则。"
```

---

## 5. Leader-GPT 输出模板类

```yaml
leader_output_templates:
  - id: "kiro-prompt-task-packet-template"
    file: "KiroPrompt-GPT-Task-Packet-Template.md"
    type: "markdown_output_template"
    output_block:
      start: "<<<KIRO_PROMPT_TASK_PACKET_START>>>"
      end: "<<<KIRO_PROMPT_TASK_PACKET_END>>>"
    use_when:
      - "生成给 KiroPrompt-GPT 的任务包。"
      - "说明本轮 target_file、file_context、任务意图、范围边界、Reading Scope、扩读条件和预期输出。"
    do_not_use_for:
      - "生成 Review-GPT 审查意图包。"
      - "生成 Round Meta JSON。"
      - "记录 Round ID、creator、recipient。"

  - id: "review-intent-packet-template"
    file: "Review-Intent-Packet-Template.md"
    type: "markdown_output_template"
    output_block:
      start: "<<<REVIEW_INTENT_PACKET_START>>>"
      end: "<<<REVIEW_INTENT_PACKET_END>>>"
    use_when:
      - "生成给 Review-GPT 的审查意图包。"
      - "说明本轮 target_file 的 Leader Intent、Review Target、Review Focus 和 Do Not Overreview。"
    do_not_use_for:
      - "生成 KiroPrompt-GPT 任务包。"
      - "替代 Review-GPT 自身审查规则。"
      - "生成 Round Meta JSON。"
      - "记录 Round ID、creator、recipient。"

  - id: "round-meta-template"
    file: "Round-Meta-Template.md"
    type: "json_output_template"
    output_block:
      start: "<<<ROUND_META_START>>>"
      end: "<<<ROUND_META_END>>>"
    use_when:
      - "生成本轮 Round Meta JSON。"
      - "记录 round_id、action、creator、output_artifacts、target_file、created_at。"
    do_not_use_for:
      - "记录相关文件。"
      - "写工作流状态说明。"
      - "写长篇背景或任务说明。"
      - "生成 KiroPrompt-GPT Task Packet 正文。"
      - "生成 Review Intent Packet 正文。"
```

---

## 6. 联合发布与发布闸门类

```yaml
release_format_files:
  - id: "leader-round-output-envelope-standard"
    file: "Leader-GPT-Round-Output-Envelope-Standard.md"
    type: "output_envelope_standard"
    priority: "high"
    use_when:
      - "将 KiroPrompt-GPT Task Packet、Review Intent Packet、Round Meta 组合成一轮正式输出。"
      - "确认三个强 marker 区块的顺序。"
      - "确认前两个区块为 Markdown，第三个区块为 JSON。"
      - "套用正式输出 envelope。"
    do_not_use_for:
      - "判断项目方向。"
      - "判断模块边界。"
      - "替代任一具体模板。"
      - "执行 release gate 检查。"

  - id: "leader-output-release-gate"
    file: "Leader-GPT-Output-Release-Gate.yaml"
    type: "release_gate"
    priority: "highest_before_output"
    use_when:
      - "Leader-GPT 输出最终内容前进行自检。"
      - "确认三个区块全部存在。"
      - "确认 marker 完整。"
      - "确认 Round Meta 是合法 JSON。"
      - "确认 Round Meta 只包含固定字段。"
      - "确认 Reading Scope 使用 R? + D?。"
      - "确认任务包、审查意图包、Round Meta 没有混写。"
      - "确认三个输出区块围绕同一个 target_file。"
    do_not_use_for:
      - "生成项目背景。"
      - "选择目标 Spec。"
      - "生成 KiroPrompt-GPT Task Packet。"
      - "生成 Review Intent Packet。"
      - "生成 Round Meta。"
      - "套 envelope。"
```

---

## 7. 信息节点路由

```yaml
information_node_routing:
  identify_current_task:
    purpose: "判断用户当前输入要求 Leader-GPT 做什么。"
    use:
      - "current-specs-completion-direction"
      - "round-meta-template"

  intake_summary_or_revision_packet:
    purpose: "处理上一轮 Summary 或大修返工包。"
    use:
      - "current-specs-completion-direction"
      - "KB-source-distillation"
      - "KB-module-knowledge-graph"

  decide_round_action:
    purpose: "判断本轮 action 是 Create 还是 MajorRevision。"
    use:
      - "round-meta-template"

  choose_target_spec_or_target_file:
    purpose: "判断本轮应新增或大修哪个目标文件。"
    use:
      - "current-specs-completion-direction"
      - "KB-module-knowledge-graph"
      - "KB-source-distillation"
      - "kiro-specs-definition"

  understand_project_background:
    purpose: "理解项目背景、模块关系和当前目标 Spec 所处位置。"
    use:
      - "KB-module-knowledge-graph"
      - "KB-source-distillation"

  build_kiro_prompt_task_packet:
    purpose: "生成 KiroPrompt-GPT Task Packet。"
    use:
      - "kiro-prompt-task-packet-template"
      - "reading-layer-quantization-standard"
      - "kiro-specs-definition"
      - "current-specs-completion-direction"
      - "KB-source-distillation"

  build_review_intent_packet:
    purpose: "生成 Review Intent Packet。"
    use:
      - "review-intent-packet-template"
      - "current-specs-completion-direction"
      - "KB-module-knowledge-graph"
      - "KB-source-distillation"

  build_round_meta:
    purpose: "生成 Round Meta JSON。"
    use:
      - "round-meta-template"

  publish_round_output:
    purpose: "将三个输出块组合为一轮正式输出。"
    use:
      - "leader-round-output-envelope-standard"

  validate_before_release:
    purpose: "最终输出前进行发布闸门检查。"
    use:
      - "leader-output-release-gate"
```

---

## 8. 默认调用顺序

```yaml
default_execution_order:
  - step: 1
    node: "identify_current_task"
    goal: "识别当前输入是用户新指令、上一轮 Summary，还是大修返工包。"

  - step: 2
    node: "intake_summary_or_revision_packet"
    goal: "如果输入是 Summary，则判断上一轮完成情况并提取对后续 Specs 的影响；如果输入是大修返工包，则保留返工目标和返工意图。"

  - step: 3
    node: "decide_round_action"
    goal: "如果输入是大修返工包或用户明确要求大修，则 action = MajorRevision；否则 action = Create。"

  - step: 4
    node: "choose_target_spec_or_target_file"
    goal: "根据 action、用户指令、Summary 或返工包确定本轮唯一 target_file。"

  - step: 5
    node: "understand_project_background"
    goal: "只读取与本轮目标相关的项目背景和模块边界。"

  - step: 6
    node: "build_kiro_prompt_task_packet"
    goal: "生成给 KiroPrompt-GPT 的 Markdown 任务包。"

  - step: 7
    node: "build_review_intent_packet"
    goal: "生成给 Review-GPT 的 Markdown 审查意图包。"

  - step: 8
    node: "build_round_meta"
    goal: "生成本轮 Round Meta JSON。"

  - step: 9
    node: "publish_round_output"
    goal: "按固定强 marker 顺序组合三个区块，套用正式输出 envelope。"

  - step: 10
    node: "validate_before_release"
    goal: "发布前检查格式、边界、必填内容、Round Meta 合法性、Reading Scope 标注和区块职责分离。"
```

---

## 9. 使用边界

```yaml
catalog_boundary:
  must:
    - "把本目录作为知识文件导航，而不是任务输出模板。"
    - "按信息节点调用文件，不把所有文件平均使用。"
    - "优先使用方向文件确认当前阶段。"
    - "优先使用模板文件生成对应输出块。"
    - "先使用 envelope 文件组合三个输出块。"
    - "最终输出前必须使用 release gate 自检。"

  must_not:
    - "把目录文件内容写进最终任务包正文。"
    - "把模板说明当作项目背景。"
    - "把项目背景文件当作输出格式约束。"
    - "把 envelope 文件当作 release gate。"
    - "把 release gate 文件当作输出模板。"
    - "跳过 Round Meta。"
    - "跳过正式输出 envelope。"
    - "跳过发布闸门。"
```

