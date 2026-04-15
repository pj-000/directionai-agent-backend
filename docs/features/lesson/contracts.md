# 教案功能契约文档

## 1. 文档定位

这份文档定义 lesson 链的各种契约：

- 外部 request / response
- SSE 事件
- 内部 TaskSpec
- Artifact
- ToolResult
- Error code

如果这些东西要变，先改本文件，再改实现。

## 2. 外部请求契约

### 2.1 兼容旧前端请求

兼容层至少应接受这些语义字段：

```json
{
  "course": "大模型算法与实践",
  "units": "强化学习",
  "lessons": "强化学习做LLM对齐学习",
  "constraint": "面向研究生，内容前沿，偏研究",
  "word_limit": 2500,
  "model_type": "qwen3.5-plus",
  "use_rag": true,
  "enable_thinking": true
}
```

### 2.2 内部 LessonTaskSpec 建议

```json
{
  "request_id": "req_xxx",
  "course": "大模型算法与实践",
  "units": ["强化学习"],
  "lessons": ["强化学习做LLM对齐学习"],
  "constraint": "面向研究生，内容前沿，偏研究",
  "word_limit": 2500,
  "use_rag": true,
  "enable_thinking": true,
  "source_documents": [],
  "model_profile": "qwen3.5-plus"
}
```

## 3. SSE 契约

### 3.1 事件类型

必须兼容：

- `thinking_start`
- `thinking_chunk`
- `thinking_end`
- `progress`
- `done`
- `error`

### 3.2 thinking_start

```json
{
  "step": 1,
  "node": "正在规划教案结构"
}
```

### 3.3 thinking_chunk

纯文本增量即可。

### 3.4 progress

```json
{
  "step": 1,
  "total": 4,
  "message": "已完成结构规划，准备生成教学设计"
}
```

### 3.5 done

最低兼容要求：

```json
{
  "lesson_plan": "# 教案标题\n\n..."
}
```

建议扩展字段：

```json
{
  "lesson_plan": "# 教案标题\n\n...",
  "artifact_id": "lesson_artifact_xxx",
  "trace_id": "trace_xxx",
  "word_count": 2380,
  "review_passed": true
}
```

### 3.6 error

```json
{
  "detail": "LESSON_SCHEMA_VALIDATION_FAILED"
}
```

## 4. Artifact 契约

## 4.1 LessonPlanOutlineArtifact

建议字段：

- `title`
- `sections`
- `estimated_word_distribution`
- `teaching_focus`

## 4.2 LessonKnowledgeContextArtifact

建议字段：

- `retrieval_enabled`
- `sources`
- `key_concepts`
- `supporting_materials`

## 4.3 LessonPlanArtifact

建议字段：

- `artifact_id`
- `course`
- `units`
- `lessons`
- `markdown_content`
- `teaching_objectives`
- `key_points`
- `difficult_points`
- `teaching_methods`
- `teaching_process`
- `activities`
- `summary`
- `assignments`
- `downstream_context`

## 5. downstream_context 契约

为了给试卷和 PPT 用，推荐至少包含：

```json
{
  "topic": "强化学习做LLM对齐学习",
  "audience": "研究生",
  "outline": [
    "研究背景",
    "RLHF基础",
    "奖励模型",
    "PPO与DPO",
    "实践案例"
  ],
  "knowledge_points": [
    "RLHF",
    "奖励模型",
    "PPO",
    "DPO"
  ],
  "activity_highlights": [
    "论文阅读",
    "案例分析"
  ]
}
```

## 6. ToolResult 契约

统一采用：

```json
{
  "success": true,
  "data": {},
  "error": null,
  "meta": {}
}
```

失败时：

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "LESSON_RAG_TIMEOUT",
    "message": "rag tool timeout"
  },
  "meta": {
    "retryable": true
  }
}
```

## 7. Skill 输出契约

lesson 相关 skill 输出尽量结构化，例如：

```json
{
  "section_name": "教学目标",
  "content_markdown": "- 理解RLHF背景\n- 掌握奖励模型作用",
  "quality_notes": []
}
```

不允许只返回一段不带结构的自由文本给下游强依赖。

## 8. 错误码建议

推荐至少定义：

- `LESSON_INVALID_REQUEST`
- `LESSON_RAG_TIMEOUT`
- `LESSON_RAG_UNAVAILABLE`
- `LESSON_DOCUMENT_PARSE_FAILED`
- `LESSON_SCHEMA_VALIDATION_FAILED`
- `LESSON_REVIEW_FAILED`
- `LESSON_INTERNAL_ERROR`

## 9. Contract Test 最低要求

至少应测试：

1. 兼容旧 request 可被正确归一化
2. `done.lesson_plan` 一定存在
3. progress 事件结构稳定
4. error 事件至少带 detail 或稳定 message
5. artifact 中 `downstream_context` 存在且结构稳定
