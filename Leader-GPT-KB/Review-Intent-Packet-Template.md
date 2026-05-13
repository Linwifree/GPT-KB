# Review-Intent-Packet-Template

用途：用于规范审查意图包的格式


格式：
**Markdown**，使用强 marker 包裹。

```text
<<<REVIEW_INTENT_PACKET_START>>>
# Review Intent Packet

...
<<<REVIEW_INTENT_PACKET_END>>>
```

## 必须包含的内容（本yaml仅为模板约束说明，具体格式参考下方的**建议 Markdown 骨架**）

```yaml
review_intent_packet_requirements:
  format: "Markdown"
  wrapped_by:
    start: "<<<REVIEW_INTENT_PACKET_START>>>"
    end: "<<<REVIEW_INTENT_PACKET_END>>>"

  required_sections:
    - "Round"
    - "Review Target"
    - "Leader Intent"
    - "Expected Alignment"
    - "Review Focus"
    - "Do Not Overreview"
    - "Optional Reading Scope"

  section_requirements:
    Round:
      must_include:
        - "Round ID"
        - "Creator: Leader-GPT"
        - "Recipient: Review-GPT"

    Review_Target:
      purpose: "说明本轮应审查哪些新生成或修改文件。"
      must_include:
        - "Target Files"
        - "Related Spec"

    Leader_Intent:
      purpose: "说明 Leader-GPT 本轮原本希望产物完成什么目标。"
      should_include:
        - "本轮生成目标"
        - "产物应解决的问题"
        - "产物不应扩展到的范围"

    Expected_Alignment:
      purpose: "说明产物应与哪些方向保持一致。"
      should_include:
        - "当前 Specs completion 方向"
        - "对应模块架构边界"
        - "相关 Specs 的前置假设"
        - "低风险底层框架 Spec 的结构假设，如本轮相关"

    Review_Focus:
      purpose: "说明审查重点。"
      should_include:
        - "是否完成本轮目标"
        - "是否保持单一工程域"
        - "是否存在跨模块大杂烩"
        - "requirements.md / design.md / tasks.md 语义是否一致"
        - "是否与已有模块边界冲突"

    Do_Not_Overreview:
      purpose: "防止 Review-GPT 过度审查。"
      must_include:
        - "不要把 tasks.md 审成 PR 计划。"
        - "不要要求改变 Kiro 原生任务块格式。"
        - "不要因为没有实现代码而判定 Spec 失败。"
        - "不要审查本轮目标之外的未来系统设计。"

    Optional_Reading_Scope:
      purpose: "仅在 Review-GPT 需要按需阅读时使用。"
      must:
        - "如果列文件，必须使用 R? + D?。"
      may:
        - "可以为空或写无。"
```

## 建议 Markdown 骨架

```markdown
<<<REVIEW_INTENT_PACKET_START>>>
# Review Intent Packet

## 1. Round
- Round ID:
- Creator: Leader-GPT
- Recipient: Review-GPT

## 2. Review Target
- Related Spec:
- Target Files:
  - `.../requirements.md`
  - `.../design.md`
  - `.../tasks.md`

## 3. Leader Intent
[说明本轮原本希望这些文件完成什么目标]

## 4. Expected Alignment
- ...
- ...

## 5. Review Focus
- ...
- ...

## 6. Do Not Overreview
- 不要把 tasks.md 审成 PR 计划。
- 不要要求改变 Kiro 原生任务块格式。
- 不要因为没有实现代码而判定 Spec 失败。
- 不要审查本轮目标之外的内容。

## 7. Optional Reading Scope
- `...`: R0 + D5（用途说明）
- `...`: R1 + D3（用途说明）
<<<REVIEW_INTENT_PACKET_END>>>
```
