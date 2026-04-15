# PPT 功能需求文档

## 1. 文档定位

这份文档定义 PPT 功能应该做什么。

如果以后发生以下变化，必须先改本文件：

- 前端可见 PPT 行为变化
- preview / done 结果语义变化
- 上传文档流程变化
- 质量门槛变化

## 2. 功能目标

PPT 链的目标是：根据课程信息、页数限制、附加要求、可选教案和文档上下文，生成一份可下载、可预览、可追踪的 PPT 结果。

它的重点不是“写出一篇内容”，而是：

- 生成 PPT 文件
- 生成前端可展示的 markdown 内容
- 生成预览图
- 提供下载地址
- 保留 trace、QA、repair 能力

## 3. 用户可见输入

根据旧系统和当前成熟 PPT 子系统，PPT 至少应支持这些语义：

- `topic`
- `course`
- `units`
- `lessons`
- `knowledge_points`
- `constraint`
- `slides`
- `page_limit`
- `language`
- `audience`
- `use_rag`
- `document_text`
- `document_name`
- `image_mode`
- `style`
- `model_provider`
- `output_language`
- `target_audience`

这些字段主要来自：

- `/Users/sss/directionai/directionai_pptagent/api.py`
- `/Users/sss/directionai/DirectionAICloud/evoagentx/evo_modules/ppt_generator.py`

## 4. 用户可见输出

最终用户至少应能看到：

- markdown 内容
- PPT 下载地址
- PPT 预览图
- 如需要，预览警告
- 生成过程

## 5. 当前前端兼容要求

旧前端 PPT store：

- `/Users/sss/directionai/DirectionAICloud/nginx/src/stores/modules/ppt-generation.ts`

当前已依赖：

### preview 阶段

- `markdown_content`
- `completed_slides`
- `total_slides`
- `current_title`

### done 阶段

- `markdown_content`
- `display_url`
- `download_url`
- `preview_images`
- `preview_warning`
- `biz_id`

这些字段语义必须继续成立。

## 6. 当前前端已依赖的流式事件

当前已依赖：

- `thinking_start`
- `thinking_chunk`
- `thinking_end`
- `progress`
- `preview`
- `done`
- `error`

PPT 是三条链里唯一明确依赖 `preview` 事件的链。

## 7. 上传文档要求

PPT 需要保留上传文档再生成的路径。

因此至少要支持：

1. 上传文档
2. 提取文档文本
3. 输出摘要 / 预览
4. 把 `document_text + document_name` 传入 PPT 生成链

## 8. 质量要求

### 8.1 质量基线

PPT 迁移后生成效果不得低于：

- `/Users/sss/directionai/directionai_pptagent`

### 8.2 不允许的退化

不允许因为“统一框架”而牺牲：

- 页面排版质量
- 预览图能力
- QA / repair 能力
- trace 能力
- render/export 能力

## 9. 与 DeerFlow 的关系

PPT 可以被 DeerFlow 编排，但不应被 DeerFlow 完全替代成自由文本 Agent。

更准确地说：

- DeerFlow 负责任务编排和上下文整理
- PPT 专用子系统负责 render / preview / QA / repair / trace

## 10. 非功能性要求

### 10.1 可追踪

至少应能定位：

- 哪次请求生成了这份 PPT
- 是否有 harness trace
- 哪一步失败

### 10.2 可回归

PPT 的任何关键改动都应可回归验证。

### 10.3 可预览

即使最终 PPT 文件可下载，系统也应尽量提供预览图。

## 11. 验收标准

1. 旧前端 PPT 页面无需大改
2. `preview` 和 `done` 字段语义稳定
3. 生成效果不低于 `directionai_pptagent` 基线
4. 保留 QA / repair / trace
5. 有 contract / integration / regression tests
