# Round-Meta-Template

用途：
要求 Leader-GPT 在每一轮输出中同时生成一段 `round meta` JSON，用于记录本轮基础元信息。
它只记录：本轮编号、本轮动作、创建者、固定输出产物与对应接收者、本轮目标文件、生成时间。
除本模板定义的 JSON 字段外，不生成其他字段。

格式：
**JSON**，使用强 marker 包裹。

```text
<<<ROUND_META_START>>>
{
  ...
}
<<<ROUND_META_END>>>
```

## 字段定义

```yaml
round_meta_json_fields:
  round_id:
    type: "number"
    meaning: "本轮工作的顺序编号，用于把同一轮中不同工具、节点、产物关联起来。"
    rule:
      - "第一轮为 1。"
      - "之后每一轮在上一轮 round_id 基础上 +1。"
      - "如果无法确认上一轮编号，则从 1 开始。"

  action:
    type: "string"
    allowed_values:
      - "Create"
      - "MajorRevision"
    meaning: "本轮 Leader-GPT 发起的动作类型。"
    value_meaning:
      Create: "本轮目标是新增一个目标文件。"
      MajorRevision: "本轮目标是针对已有目标文件进行大修返工。"

  creator:
    type: "string"
    fixed_value: "Leader-GPT"
    meaning: "本轮 round meta 的创建者。"

  output_artifacts:
    type: "array"
    fixed_value:
      - type: "kiro_prompt_task_packet"
        recipient: "KiroPrompt-GPT"
      - type: "review_intent_packet"
        recipient: "Review-GPT"
    meaning: "Leader-GPT 本轮固定输出的两个主要产物，以及每个产物对应的下游接收者。"

  target_file:
    type: "string"
    meaning: "本轮打算新增或大修的唯一目标文件路径。一次默认只处理一个目标文件。"

  created_at:
    type: "string"
    format: "ISO 8601"
    meaning: "本轮 round meta 的生成时间。"
```

## 建议 JSON 骨架

```json
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
```

## Round Meta 输出示例

```text
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
  "target_file": ".kiro/specs/auth-role-permission/requirements.md",
  "created_at": "2026-05-11T00:00:00Z"
}
<<<ROUND_META_END>>>
```
