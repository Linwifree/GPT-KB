# Leader-GPT Round Output Envelope Standard

用途：
本文件用于约束 Leader-GPT 在**一轮对话输出**中如何同时发布以下三类产物：

1. `KiroPrompt-GPT Task Packet`
2. `Review Intent Packet`
3. `Round Meta`

目标是保证一轮输出结构稳定、强 marker 可清洗、三个区块职责分离，并且不把不同接收对象的内容混写。

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
      - "Leader-GPT 每轮输出必须按固定顺序发布三个区块。"
      - "第一个区块必须是 KiroPrompt-GPT Task Packet。"
      - "第二个区块必须是 Review Intent Packet。"
      - "第三个区块必须是 Round Meta。"
    must_not:
      - "不得调换三个区块顺序。"
      - "不得省略任一区块。"
      - "不得把多个区块合并成一个区块。"
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
{
  "round_id": 1,
  "action": "Create",
  "creator": "Leader-GPT",
  "output_artifacts": [
    {
      "type": "kiro_prompt_task_packet",
      "recipient": "KiroPrompt-GPT"
    },
    {
      "type": "review_intent_packet",
      "recipient": "Review-GPT"
    }
  ],
  "target_file": ".kiro/specs/<spec-name>/<target-file>.md",
  "created_at": "YYYY-MM-DDTHH:MM:SSZ"
}
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
      - "说明目标 Spec、任务意图、范围边界、阅读范围与预期输出。"
    must_not:
      - "不得写 Review-GPT 的审查意图。"
      - "不得写 round meta JSON。"
      - "不得解释 Review-GPT 如何审查。"

  review_intent_packet:
    recipient: "Review-GPT"
    format: "Markdown"
    purpose:
      - "传递本轮审查意图。"
      - "说明本轮产物应完成什么、审查重点是什么、不要过度审查什么。"
    must_not:
      - "不得写 KiroPrompt-GPT 的任务执行细节。"
      - "不得写 prompt-kiro.md 生成步骤。"
      - "不得写 round meta JSON。"

  round_meta:
    format: "JSON"
    purpose:
      - "记录本轮基础元信息。"
      - "只记录 round_id、action、creator、output_artifacts、target_file、created_at。"
    must_not:
      - "不得写任务包正文。"
      - "不得写审查意图正文。"
      - "不得写本模板未定义的额外 JSON 字段。"
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
    - "不得在 marker 外补充影响任务理解的正文。"
    - "不得在 marker 外写第四类产物。"
    - "不得在 marker 外写与本轮任务无关的背景说明。"
```

---

## 6. Round Meta 与前两个区块的对应关系

```yaml
round_meta_alignment:
  must:
    - "round_meta.round_id 必须与 KiroPrompt-GPT Task Packet 和 Review Intent Packet 中的 Round ID 一致。"
    - "round_meta.action 必须反映本轮动作类型。"
    - "round_meta.target_file 必须是本轮打算新增或大修的唯一目标文件。"
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
  must:
    - "target_file 使用 string。"
    - "target_file 不使用数组。"
    - "target_file 不记录泛泛相关文件。"
  must_not:
    - "不得在 round_meta 中同时列出 requirements.md、design.md、tasks.md 三个示例目标。"
    - "不得把阅读范围文件写入 target_file。"
    - "不得把相关文件、参考文件、依赖文件写入 target_file。"
```

---

## 8. 发布前自检规则

```yaml
release_check:
  pass_when:
    - "三个区块全部存在。"
    - "三个区块顺序正确。"
    - "所有 start marker 与 end marker 完整且拼写一致。"
    - "KiroPrompt-GPT Task Packet 是 Markdown。"
    - "Review Intent Packet 是 Markdown。"
    - "Round Meta 是合法 JSON。"
    - "Round Meta 只包含本模板定义的字段。"
    - "Round Meta 的 output_artifacts 与固定值一致。"
    - "Round Meta 的 action 只能是 Create 或 MajorRevision。"
    - "Round Meta 的 target_file 是单个目标文件路径。"

  fail_when:
    - "缺失任一区块。"
    - "强 marker 缺失、拼写错误或顺序错乱。"
    - "Round Meta 不是合法 JSON。"
    - "Round Meta 出现模板外字段。"
    - "Round Meta 使用 target_files 数组。"
    - "Round Meta 把相关文件写成目标文件。"
    - "KiroPrompt-GPT Task Packet 与 Review Intent Packet 内容混写。"
```

---

## 9. 最终输出形态要求

```yaml
final_output_requirement:
  must:
    - "Leader-GPT 一轮输出必须由三个固定强 marker 区块组成。"
    - "前两个区块使用 Markdown。"
    - "第三个区块使用 JSON。"
    - "三个区块分别服务不同下游对象，不互相污染。"
    - "Round Meta 保持最小字段集合。"

  must_not:
    - "不得新增第四个输出区块。"
    - "不得把 YAML 作为 Round Meta 的输出格式。"
    - "不得把 Round Meta 写成解释文本。"
    - "不得在 Round Meta 中添加本模板未定义字段。"
```
