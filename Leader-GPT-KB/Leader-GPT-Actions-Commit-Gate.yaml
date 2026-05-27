system_facts:
  file_identity:
    id: "leader-gpt-actions-commit-gate"
    name: "Leader-GPT Actions Commit Gate"
    owner_role: "Leader-GPT"
    use_stage: "before_commitLeaderArtifacts"
    purpose: "定义 Leader-GPT 调用 commitLeaderArtifacts 前的提交体自检规则。"

  role_scope:
    owns:
      - "确认本轮已读取 Gateway context。"
      - "确认 requestBody 顶层字段集合正确。"
      - "确认 task_packet_md 是任务包 Markdown 正文。"
      - "确认 review_intent_pack_md 是审查意图 Markdown 正文。"
      - "确认 leader_work_summary_json 是 JSON object。"
      - "确认三个产物围绕同一个 target_file。"
      - "确认 action 与 file_change.type 配对正确。"

    out_of_scope:
      - "选择本轮 target_file。"
      - "生成 task_packet_md 正文。"
      - "生成 review_intent_pack_md 正文。"
      - "生成 leader_work_summary_json 字段内容。"
      - "定义 Markdown 正文的章节语义。"
      - "定义 Gateway 实现细节。"
      - "定义其他角色的系统规则。"

source_contracts:
  gateway_contract: "Leader-GPT-Actions-Gateway-Contract.md"
  commit_payload_format: "Leader-GPT-Commit-Payload-Format.yaml"
  task_packet_template: "KiroPrompt-GPT-Task-Packet-Template.md"
  review_intent_template: "Review-Intent-Packet-Template.md"
  work_summary_template: "Leader-Work-Summary-Template.yaml"

commit_gate_contract:
  required_context_fields:
    - "mode"
    - "input_type"
    - "expected_action"
    - "context.current_state"
    - "context.current_input_md"
    - "context.optional_context"

  mode_values:
    - "first-round"
    - "received-summary"
    - "rework"

  expected_action_values:
    - "Create"
    - "MajorRevision"

  request_body:
    type: "JSON object"
    exact_top_level_fields:
      - "task_packet_md"
      - "review_intent_pack_md"
      - "leader_work_summary_json"

  field_roles:
    task_packet_md:
      type: "multiline Markdown string"
      document_role: "KiroPrompt-GPT task packet body"
      template_source: "KiroPrompt-GPT-Task-Packet-Template.md"

    review_intent_pack_md:
      type: "multiline Markdown string"
      document_role: "Review-GPT review intent body"
      template_source: "Review-Intent-Packet-Template.md"

    leader_work_summary_json:
      type: "JSON object"
      schema_source: "Leader-Work-Summary-Template.yaml"
      canonical_source_for:
        - "target_file"
        - "action"
        - "file_change.type"
        - "dependencies"
        - "memory_note"

check_groups:
  context_acquisition_check:
    required:
      - "本轮已获得 getLeaderContext 返回值。"
      - "mode 来自 getLeaderContext.mode。"
      - "input_type 来自 getLeaderContext.input_type。"
      - "expected_action 来自 getLeaderContext.expected_action。"
      - "context.current_state 已读取。"
      - "context.current_input_md 已读取。"
      - "context.optional_context 已读取。"

    pass_when:
      - "required_context_fields 全部可用。"
      - "mode 属于 mode_values。"
      - "expected_action 属于 expected_action_values。"

    fail_when:
      - "required_context_fields 缺项。"
      - "mode 不属于 mode_values。"
      - "expected_action 不属于 expected_action_values。"

  request_body_shape_check:
    required:
      - "requestBody 是 JSON object。"
      - "requestBody 顶层字段集合等于 exact_top_level_fields。"
      - "task_packet_md 字段存在。"
      - "review_intent_pack_md 字段存在。"
      - "leader_work_summary_json 字段存在。"

    pass_when:
      - "requestBody 顶层字段集合完整且无偏移。"
      - "三个字段分别对应 field_roles 中定义的类型。"

    fail_when:
      - "requestBody 类型异常。"
      - "requestBody 顶层字段集合与 exact_top_level_fields 不一致。"
      - "task_packet_md 字段缺项。"
      - "review_intent_pack_md 字段缺项。"
      - "leader_work_summary_json 字段缺项。"

  task_packet_md_check:
    required:
      - "task_packet_md 是 multiline Markdown string。"
      - "task_packet_md 字段值是完整任务包正文。"
      - "task_packet_md 第一行是 `# KiroPrompt-GPT Task Packet`。"
      - "task_packet_md 保留 Markdown 标题、列表项和段落的真实换行。"
      - "task_packet_md 使用 KiroPrompt-GPT-Task-Packet-Template.md 定义的正文结构。"
      - "task_packet_md 包含唯一 Target File。"
      - "task_packet_md 的 Target File 与 canonical target_file 一致。"
      - "task_packet_md 的 Reading Scope 文件使用 R? + D? 标注。"
      - "task_packet_md 正文聚焦给 KiroPrompt-GPT 的任务包。"

    pass_when:
      - "task_packet_md 可作为 task-packet.md 正文。"
      - "task_packet_md 第一行符合模板标题。"
      - "task_packet_md Target File 与 canonical target_file 一致。"
      - "task_packet_md 正文职责与 task_packet_template 一致。"

    fail_when:
      - "task_packet_md 类型异常。"
      - "task_packet_md 第一行与模板标题不一致。"
      - "task_packet_md 正文结构与 task_packet_template 不一致。"
      - "task_packet_md Target File 缺项。"
      - "task_packet_md 出现多个直接 Target File。"
      - "task_packet_md Target File 与 canonical target_file 不一致。"
      - "task_packet_md Reading Scope 标注缺项。"
      - "task_packet_md 正文职责偏移。"

  review_intent_pack_md_check:
    required:
      - "review_intent_pack_md 是 multiline Markdown string。"
      - "review_intent_pack_md 字段值是完整审查意图正文。"
      - "review_intent_pack_md 第一行是 `# Review Intent Packet`。"
      - "review_intent_pack_md 保留 Markdown 标题、列表项和段落的真实换行。"
      - "review_intent_pack_md 使用 Review-Intent-Packet-Template.md 定义的正文结构。"
      - "review_intent_pack_md 包含唯一 Review Target。"
      - "review_intent_pack_md 的 Review Target 与 canonical target_file 一致。"
      - "review_intent_pack_md 的 Optional Reading Scope 如列文件，则使用 R? + D? 标注。"
      - "review_intent_pack_md 正文聚焦给 Review-GPT 的审查意图。"

    pass_when:
      - "review_intent_pack_md 可作为 review-intent-pack.md 正文。"
      - "review_intent_pack_md 第一行符合模板标题。"
      - "review_intent_pack_md Review Target 与 canonical target_file 一致。"
      - "review_intent_pack_md 正文职责与 review_intent_template 一致。"

    fail_when:
      - "review_intent_pack_md 类型异常。"
      - "review_intent_pack_md 第一行与模板标题不一致。"
      - "review_intent_pack_md 正文结构与 review_intent_template 不一致。"
      - "review_intent_pack_md Review Target 缺项。"
      - "review_intent_pack_md 出现多个直接 Review Target。"
      - "review_intent_pack_md Review Target 与 canonical target_file 不一致。"
      - "review_intent_pack_md Optional Reading Scope 标注缺项。"
      - "review_intent_pack_md 正文职责偏移。"

  leader_work_summary_json_check:
    required:
      - "leader_work_summary_json 是 JSON object。"
      - "leader_work_summary_json.schema_version = 1.0。"
      - "leader_work_summary_json.action 属于 expected_action_values。"
      - "leader_work_summary_json.action 等于 expected_action。"
      - "leader_work_summary_json.file_change.type 与 action 配对。"
      - "leader_work_summary_json.file_change.target_file 是单一 string。"
      - "leader_work_summary_json.main_goal 非空。"
      - "leader_work_summary_json.behavior.task_type 非空。"
      - "leader_work_summary_json.behavior.operation 非空。"
      - "leader_work_summary_json.boundary.in_scope 是 string[]。"
      - "leader_work_summary_json.boundary.out_of_scope 是 string[]。"
      - "leader_work_summary_json.dependencies.modules 是 string[]。"
      - "leader_work_summary_json.dependencies.files 是 string[]。"
      - "leader_work_summary_json.memory_note.summary 非空。"
      - "leader_work_summary_json.memory_note.carry_forward 是 string[]。"
      - "leader_work_summary_json 字段集合符合 Leader-Work-Summary-Template.yaml。"

    pass_when:
      - "leader_work_summary_json 符合 work_summary_template。"
      - "leader_work_summary_json.action 与 expected_action 一致。"
      - "leader_work_summary_json.file_change.type 与 action 配对正确。"
      - "leader_work_summary_json.file_change.target_file 可作为 canonical target_file。"

    fail_when:
      - "leader_work_summary_json 类型异常。"
      - "leader_work_summary_json required 字段缺项。"
      - "leader_work_summary_json 字段集合与 work_summary_template 不一致。"
      - "leader_work_summary_json.action 与 expected_action 不一致。"
      - "leader_work_summary_json.file_change.type 与 action 配对异常。"
      - "leader_work_summary_json.file_change.target_file 类型异常。"

  mode_action_alignment_check:
    canonical_pairs:
      first-round:
        expected_action: "Create"
        file_change_type: "new_file"

      received-summary:
        expected_action: "Create"
        file_change_type: "new_file"

      rework:
        expected_action: "MajorRevision"
        file_change_type: "major_revision"

    required:
      - "mode 与 expected_action 符合 canonical_pairs。"
      - "leader_work_summary_json.file_change.type 与 canonical_pairs 当前分支一致。"

    pass_when:
      - "mode、expected_action、file_change.type 三者配对一致。"

    fail_when:
      - "mode 与 expected_action 配对异常。"
      - "expected_action 与 file_change.type 配对异常。"

  cross_artifact_alignment_check:
    canonical_values:
      target_file: "leader_work_summary_json.file_change.target_file"
      action: "leader_work_summary_json.action"
      file_change_type: "leader_work_summary_json.file_change.type"

    required:
      - "task_packet_md 的 Target File 与 canonical target_file 一致。"
      - "review_intent_pack_md 的 Review Target 与 canonical target_file 一致。"
      - "task_packet_md 的 Expected Action 与 canonical action 一致。"
      - "review_intent_pack_md 的 Expected Action 与 canonical action 一致。"
      - "三个产物表达同一轮任务。"

    pass_when:
      - "Target File / Review Target / canonical target_file 一致。"
      - "两个 Markdown 正文中的 Expected Action 与 canonical action 一致。"
      - "三个产物语义属于同一 target_file。"

    fail_when:
      - "Target File 与 canonical target_file 不一致。"
      - "Review Target 与 canonical target_file 不一致。"
      - "Expected Action 与 canonical action 不一致。"
      - "三个产物语义分叉。"

  artifact_role_check:
    required:
      - "task_packet_md 承载 KiroPrompt-GPT 任务包正文。"
      - "review_intent_pack_md 承载 Review-GPT 审查意图正文。"
      - "leader_work_summary_json 承载机器可读摘要。"

    pass_when:
      - "三个产物职责清晰。"
      - "Markdown 正文与各自模板职责一致。"
      - "机器可读字段位于 leader_work_summary_json。"

    fail_when:
      - "task_packet_md 职责偏移。"
      - "review_intent_pack_md 职责偏移。"
      - "leader_work_summary_json 不是机器可读摘要对象。"
      - "机器可读主事实分散到多个产物中。"

pass_when:
  - "getLeaderContext 返回的 required_context_fields 全部可用。"
  - "requestBody 顶层字段集合等于 exact_top_level_fields。"
  - "task_packet_md 是完整任务包 Markdown 正文。"
  - "review_intent_pack_md 是完整审查意图 Markdown 正文。"
  - "leader_work_summary_json 是符合模板的 JSON object。"
  - "leader_work_summary_json.action 等于 expected_action。"
  - "leader_work_summary_json.file_change.type 与 action 配对正确。"
  - "leader_work_summary_json.file_change.target_file 是 canonical target_file。"
  - "task_packet_md 的 Target File 与 canonical target_file 一致。"
  - "review_intent_pack_md 的 Review Target 与 canonical target_file 一致。"
  - "三个产物职责清晰且属于同一轮任务。"

fail_when:
  - "context 缺项。"
  - "requestBody 形态异常。"
  - "task_packet_md 形态异常。"
  - "review_intent_pack_md 形态异常。"
  - "leader_work_summary_json 形态异常。"
  - "mode、expected_action、file_change.type 配对异常。"
  - "Target File / Review Target / canonical target_file 对齐异常。"
  - "产物职责偏移。"

stop_when_failed:
  - "停止调用 commitLeaderArtifacts。"
  - "定位失败 check_group。"
  - "修正对应产物或 requestBody。"
  - "重新执行 Actions Commit Gate。"

commit_permission:
  allow_commit_when:
    - "pass_when 全部满足。"
    - "fail_when 全部未触发。"

  block_commit_when:
    - "任一 fail_when 被触发。"
    - "无法确认 canonical target_file。"
    - "无法确认 expected_action 与 action 一致。"
    - "无法确认三个产物属于同一轮任务。"

after_commit_response_policy:
  if_gateway_accepted_true:
    chat_response:
      - "返回极简成功回执。"
      - "可简要说明本轮三个产物已提交。"

  if_gateway_accepted_false:
    chat_response:
      - "报告 Gateway 返回的 error。"
      - "报告 Gateway 返回的 message。"
      - "根据错误所属字段修正后重新执行 commit gate。"