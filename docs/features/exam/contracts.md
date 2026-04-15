# 试卷功能契约文档

## 1. 外部请求契约

### 1.1 兼容旧请求示例

```json
{
  "subject": "DEEPSEEK入门到精通",
  "knowledge_bases": "AI辅助的三重角色",
  "constraint": "",
  "language": "Chinese",
  "single_choice_num": 3,
  "multiple_choice_num": 3,
  "true_false_num": 3,
  "fill_blank_num": 2,
  "short_answer_num": 2,
  "programming_num": 1,
  "easy_percentage": 30,
  "medium_percentage": 50,
  "hard_percentage": 20,
  "use_rag": true,
  "model_type": "DeepSeek",
  "enable_thinking": false
}
```

### 1.2 内部 ExamTaskSpec 建议

```json
{
  "request_id": "req_xxx",
  "subject": "DEEPSEEK入门到精通",
  "knowledge_points": ["AI辅助的三重角色"],
  "constraint": "",
  "language": "Chinese",
  "question_counts": {
    "single_choice": 3,
    "multiple_choice": 3,
    "true_false": 3,
    "fill_blank": 2,
    "short_answer": 2,
    "programming": 1
  },
  "difficulty_distribution": {
    "easy": 30,
    "medium": 50,
    "hard": 20
  },
  "use_rag": true,
  "enable_thinking": false
}
```

## 2. SSE 契约

### 2.1 事件类型

- `thinking_start`
- `thinking_chunk`
- `thinking_end`
- `progress`
- `done`
- `error`

### 2.2 thinking_chunk 特殊兼容

当前旧前端 exam store 能识别这种 JSON chunk：

```json
{
  "step": 2,
  "text": "正在生成单选题..."
}
```

因此推荐继续兼容这种输出方式。

### 2.3 done 最低兼容要求

至少要能让前端解析出题目数组。

推荐：

```json
{
  "questions": [
    {
      "type": "single_choice",
      "question": "以下哪个属于...",
      "options": ["A", "B", "C", "D"],
      "answer": "A",
      "analysis": "因为...",
      "difficulty": "easy",
      "knowledge_point": "AI辅助的三重角色"
    }
  ]
}
```

兼容旧前端时，可同时提供：

```json
{
  "question": [...],
  "questions": [...],
  "output": {
    "question": [...]
  }
}
```

## 3. Artifact 契约

### 3.1 ExamBlueprintArtifact

字段建议：

- `subject`
- `knowledge_points`
- `question_type_plan`
- `difficulty_plan`
- `coverage_targets`

### 3.2 QuestionBatchArtifact

字段建议：

- `question_type`
- `questions`
- `generation_notes`

### 3.3 ExamPaperArtifact

字段建议：

- `artifact_id`
- `questions`
- `question_type_counts`
- `difficulty_summary`
- `coverage_summary`
- `checker_results`

## 4. Question 对象最低字段

每道题建议至少包含：

- `type`
- `question`
- `answer`
- `analysis`
- `difficulty`
- `knowledge_point`

某些题型需要额外字段：

### 单选 / 多选

- `options`

### 判断

- `answer`

### 编程题

- `reference_solution`
- `input_constraints`
- `output_constraints`

## 5. ToolResult 契约

统一：

```json
{
  "success": true,
  "data": {},
  "error": null,
  "meta": {}
}
```

## 6. Error Code 建议

- `EXAM_INVALID_REQUEST`
- `EXAM_BLUEPRINT_FAILED`
- `EXAM_GENERATOR_FAILED`
- `EXAM_CHECKER_FAILED`
- `EXAM_DEDUP_FAILED`
- `EXAM_SCHEMA_VALIDATION_FAILED`
- `EXAM_INTERNAL_ERROR`

## 7. Contract Test 最低要求

至少测试：

1. legacy request 到 ExamTaskSpec 的映射
2. `thinking_chunk` 的 JSON 路由兼容
3. `done.questions` 存在
4. 同时兼容 `question/questions/output.question`
5. 每题至少有 type / question / answer / analysis
