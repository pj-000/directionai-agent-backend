# PPT 功能契约文档

## 1. 外部请求契约

### 1.1 兼容旧请求示例

```json
{
  "topic": "",
  "course": "大模型算法与实践",
  "units": ["强化学习"],
  "lessons": ["强化学习做LLM对齐学习"],
  "knowledge_points": ["RLHF", "奖励模型", "PPO", "DPO"],
  "constraint": "面向研究生，内容前沿，偏研究",
  "slides": "10-12",
  "page_limit": null,
  "language": "中文",
  "audience": "研究生",
  "use_rag": true,
  "document_text": "",
  "document_name": ""
}
```

### 1.2 内部 PPTTaskSpec 建议

```json
{
  "request_id": "req_xxx",
  "topic": "大模型算法与实践 - 强化学习做LLM对齐学习",
  "course": "大模型算法与实践",
  "units": ["强化学习"],
  "lessons": ["强化学习做LLM对齐学习"],
  "knowledge_points": ["RLHF", "奖励模型", "PPO", "DPO"],
  "constraint": "面向研究生，内容前沿，偏研究",
  "output_language": "中文",
  "target_audience": "研究生",
  "min_slides": 10,
  "max_slides": 12,
  "enable_web_search": true,
  "document_text": "",
  "document_name": "",
  "image_mode": "auto"
}
```

## 2. SSE 契约

### 2.1 事件类型

- `thinking_start`
- `thinking_chunk`
- `thinking_end`
- `progress`
- `preview`
- `done`
- `error`

### 2.2 preview 契约

当前前端已消费的 preview 字段：

```json
{
  "markdown_content": "## 第1页 ...",
  "completed_slides": 3,
  "total_slides": 10,
  "current_title": "奖励模型"
}
```

### 2.3 done 契约

当前前端已消费的 done 字段：

```json
{
  "markdown_content": "## 全部PPT内容 ...",
  "display_url": "/download_ppt/demo.pptx",
  "download_url": "/download_ppt/demo.pptx",
  "preview_images": [
    "/preview_ppt/demo.pptx/slide_1.png"
  ],
  "preview_warning": "",
  "biz_id": "biz_xxx"
}
```

建议扩展字段：

```json
{
  "harness_trace_available": true,
  "harness_trace_path": "/debug/harness/trace/demo.json"
}
```

### 2.4 error 契约

```json
{
  "detail": "PPT_RENDER_FAILED"
}
```

## 3. Upload Document 契约

### 3.1 输入

`multipart/form-data`

- `file`

### 3.2 输出

建议至少包含：

```json
{
  "success": true,
  "document_name": "讲义.pdf",
  "document_text": "......",
  "summary": {
    "title": "讲义摘要",
    "sections": []
  }
}
```

## 4. Artifact 契约

### 4.1 PPTGenerationRequestArtifact

建议字段：

- `topic`
- `context_sources`
- `slide_bounds`
- `style`
- `image_mode`

### 4.2 PPTGenerationResultArtifact

建议字段：

- `artifact_id`
- `output_filename`
- `output_path`
- `markdown_content`
- `preview_images`
- `preview_warning`
- `download_url`
- `biz_id`

### 4.3 PPTTraceArtifact

建议字段：

- `trace_id`
- `trace_path`
- `stages`
- `tool_invocations`
- `repair_records`

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

- `PPT_INVALID_REQUEST`
- `PPT_TOPIC_MISSING`
- `PPT_DOCUMENT_PARSE_FAILED`
- `PPT_OUTLINE_FAILED`
- `PPT_RENDER_FAILED`
- `PPT_PREVIEW_GENERATION_FAILED`
- `PPT_QA_FAILED`
- `PPT_REPAIR_FAILED`
- `PPT_INTERNAL_ERROR`

## 7. Contract Test 最低要求

至少测试：

1. legacy compatibility fields 能正确归一化
2. preview 字段结构稳定
3. done 字段结构稳定
4. download / preview url 可生成
5. upload_document 输出可用于 stream_ppt
6. trace 字段存在时不会破坏前端兼容
