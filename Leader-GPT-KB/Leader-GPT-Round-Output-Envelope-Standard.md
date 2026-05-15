# Leader-GPT Round Output Envelope Standard

用途：
本文件用于约束 Leader-GPT 在**一轮正式输出**中如何套用固定 envelope，同时发布以下三类产物：

1. `KiroPrompt-GPT Task Packet`
2. `Review Intent Packet`
3. `Round Meta`

目标是保证一轮输出结构稳定、强 marker 可清洗、三个区块职责分离，并且不把不同接收对象的内容混写。

本文件只负责“套 envelope”。  
正式输出前是否允许发布，由 `Leader-GPT-Output-Release-Gate.yaml` 负责检查。

---

## 1. 一轮输出的固定顺序

```yaml
round_output_envelope:
  output_order:
    - "kiro_prompt_task_packet"
    - "review_intent_packet"
    - "round_meta"

  order_rule:
    must:
      - "Leader-GPT 每轮正式输出必须按固定顺序发布三个区块。"
      - "第一个区块必须是 KiroPrompt-GPT Task Packet。"
      - "第二个区块必须是 Review Intent Packet。"
      - "第三个区块必须是 Round Meta。"

    must_not:
      - "调换三个区块顺序。"
      - "省略任一区块。"
      - "把多个区块合并成一个区块。"
      - "新增第四个正式输出区块。"
```

---

## 2. 固定强 marker

```yaml
strong_markers:
  kiro_prompt_task_packet:
    format: "Markdown"
    start: "<<<KIRO_PROMPT_TASK_PACKET_START>>>"
    end: "<<<KIRO_PROMPT_TASK_PACKET_END>>>"

  review_intent_packet:
    format: "Markdown"
    start: "<<<REVIEW_INTENT_PACKET_START>>>"
    end: "<<<REVIEW_INTENT_PACKET_END>>>"

  round_meta:
    format: "JSON"
    start: "<<<ROUND_META_START>>>"
    end: "<<<ROUND_META_END>>>"
```

---

## 3. 标准输出骨架

```text
<<<KIRO_PROMPT_TASK_PACKET_START>>>
# KiroPrompt-GPT Task Packet

[这里放给 KiroPrompt-GPT 的 Markdown 任务包]
<<<KIRO_PROMPT_TASK_PACKET_END>>>

<<<REVIEW_INTENT_PACKET_START>>>
# Review Intent Packet

[这里放给 Review-GPT 的 Markdown 审查意图包]
<<<REVIEW_INTENT_PACKET_END>>>

<<<ROUND_META_START>>>
[这里放由 Round-Meta-Template.md 生成的合法 JSON]
<<<ROUND_META_END>>>
```

---

## 4. 三个区块的职责边界

```yaml
block_boundary_rules:
  kiro_prompt_task_packet:
    recipient: "KiroPrompt-GPT"
    format: "Markdown"
    purpose:
      - "传递本轮 Kiro prompt 生成任务。"
      - "说明本轮 target_file、file_context、任务意图、范围边界、Reading Scope、扩读许可和预期输出。"
    must_not:
      - "写 Review-GPT 的审查意图。"
      - "写 Review-GPT 的完整审查制度。"
      - "写 Round Meta JSON。"
      - "解释 Review-GPT 如何审查。"

  review_intent_packet:
    recipient: "Review-GPT"
    format: "Markdown"
    purpose:
      - "传递本轮审查意图。"
      - "说明本轮 target_file 的审查目标、Leader Intent、审查重点和不要过度审查的内容。"
    must_not:
      - "写 KiroPrompt-GPT 的任务执行细节。"
      - "写 prompt-kiro.md 生成步骤。"
      - "写 Round Meta JSON。"
      - "定义 Review-GPT 的完整审查制度。"

  round_meta:
    format: "JSON"
    purpose:
      - "记录本轮基础元信息。"
      - "只记录 round_id、action、creator、output_artifacts、target_file、created_at。"
    must_not:
      - "写任务包正文。"
      - "写审查意图正文。"
      - "写本模板未定义的额外 JSON 字段。"
      - "写工作流状态说明、日志、备注或相关文件列表。"
```

---

## 5. 区块外内容规则

```yaml
outside_block_policy:
  must:
    - "正式输出时，三个强 marker 区块应构成主要输出内容。"
    - "自动化程序只依赖 marker 内部内容。"

  should:
    - "除非用户明确要求解释，否则不要在三个区块外写额外说明。"

  must_not:
    - "在 marker 外补充影响任务理解的正文。"
    - "在 marker 外写第四类产物。"
    - "在 marker 外写与本轮任务无关的背景说明。"
    - "在 marker 外写会被下游误认为任务内容的补充说明。"
```

---

## 6. Round Meta 与前两个区块的对应关系

```yaml
round_meta_alignment:
  must:
    - "round_meta.action 必须反映本轮动作类型。"
    - "round_meta.target_file 必须是本轮打算新增或大修的唯一目标文件。"
    - "round_meta.target_file 必须与 KiroPrompt-GPT Task Packet 中的 Target File 一致。"
    - "round_meta.target_file 必须与 Review Intent Packet 中的 Review Target 一致。"
    - "round_meta.output_artifacts 必须固定对应前两个 Markdown 区块。"

  action_allowed_values:
    - "Create"
    - "MajorRevision"

  output_artifacts_fixed_value:
    - type: "kiro_prompt_task_packet"
      recipient: "KiroPrompt-GPT"
    - type: "review_intent_packet"
      recipient: "Review-GPT"
```

---

## 7. 单目标文件规则

```yaml
single_target_file_policy:
  principle:
    - "一轮默认只面向一个主要目标文件。"
    - "round_meta.target_file 只记录本轮打算新增或大修的目标文件。"
    - "相关文件、参考文件、阅读文件只应出现在任务包或审查意图包中，不进入 Round Meta target_file。"

  must:
    - "target_file 使用 string。"
    - "target_file 记录本轮唯一目标文件路径。"
    - "target_file 与 KiroPrompt-GPT Task Packet 中的本轮目标文件保持一致。"
    - "target_file 与 Review Intent Packet 中的审查目标保持一致。"

  must_not:
    - "使用 target_files 数组。"
    - "在 round_meta 中同时列出 requirements.md、design.md、tasks.md 三个目标文件。"
    - "把阅读范围文件写入 target_file。"
    - "把相关文件、参考文件、依赖文件写入 target_file。"
```

---

## 8. 最终输出形态要求

```yaml
final_output_requirement:
  must:
    - "Leader-GPT 一轮正式输出必须由三个固定强 marker 区块组成。"
    - "前两个区块使用 Markdown。"
    - "第三个区块使用 JSON。"
    - "三个区块分别服务不同下游对象，不互相污染。"
    - "Round Meta 保持最小字段集合。"
    - "套用 envelope 后，必须再执行 `Leader-GPT-Output-Release-Gate.yaml`。"

  must_not:
    - "新增第四个正式输出区块。"
    - "把 YAML 作为 Round Meta 的输出格式。"
    - "把 Round Meta 写成解释文本。"
    - "在 Round Meta 中添加本模板未定义字段。"
```

---

## 9. 与 Release Gate 的关系

```yaml
relationship_to_release_gate:
  current_file:
    role: "套 envelope"
    answers:
      - "三个输出块应该按什么顺序出现？"
      - "每个输出块用什么 marker？"
      - "每个输出块使用什么格式？"
      - "三个输出块之间如何保持职责分离？"

  release_gate_file:
    file: "Leader-GPT-Output-Release-Gate.yaml"
    role: "过 release gate"
    answers:
      - "当前输出是否满足发布条件？"
      - "三个区块是否完整、顺序正确、marker 正确？"
      - "Round Meta 是否合法且字段最小？"
      - "Reading Scope 是否使用 R? + D?。"
      - "任务包、审查意图包、Round Meta 是否混写？"

  execution_order:
    - "先生成 KiroPrompt-GPT Task Packet。"
    - "再生成 Review Intent Packet。"
    - "再生成 Round Meta。"
    - "然后使用本文件套 envelope。"
    - "最后使用 `Leader-GPT-Output-Release-Gate.yaml` 进行发布前检查。"
```