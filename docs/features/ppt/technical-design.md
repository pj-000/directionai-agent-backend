# PPT 功能技术设计文档

## 1. 设计原则

PPT 链的重构目标不是“把所有逻辑统一成 DeerFlow Agent”，而是：

1. 用 DeerFlow 接管编排层
2. 保留 PPT 专用子系统的成熟工程能力
3. 通过 Compatibility API 对接旧前端

## 2. 主基线来源

PPT 子系统的主基线来自：

- `/Users/sss/directionai/directionai_pptagent`

而不是旧的：

- `/Users/sss/directionai/DirectionAICloud/pptagent`
- `/Users/sss/directionai/DirectionAICloud/evoagentx/evo_modules/ppt_generator.py`

旧代码的作用主要是：

- 参考旧前端兼容字段
- 参考旧联调方式

## 3. 当前仓库目标落点

推荐目录：

```text
backend/packages/directionai/ppt/
├─ ppt_schemas.py
├─ ppt_request_builder.py
├─ ppt_subsystem_adapter.py
├─ ppt_result_mapper.py
└─ ppt_trace_service.py
```

配套路由：

```text
backend/app/gateway/routers/ppt_router.py
backend/app/gateway/routers/compatibility_router.py
```

## 4. DeerFlow Sub-Agent 策略

PPT 链明确不把 DeerFlow 动态生成 subagent 作为核心实现方式。

### 4.1 PPT 链到底有没有用到 DeerFlow 生成 subagent

对于核心 PPT 生成流程，答案是没有。

更准确地说：

- DeerFlow 可以负责是否生成 PPT、如何整理上游上下文、何时调用 PPT 子系统
- DeerFlow 不负责现场孵化页面写手、排版师、视觉修复师之类的新 subagent 来替代专用 PPT runtime

### 4.2 为什么 PPT 链不能按 lesson / exam 的方式处理

因为 PPT 的核心难点不是角色分工抽象，而是：

- 渲染正确性
- 模板和版式稳定性
- 预览图生成
- QA / repair / trace / benchmark 闭环

这些都更依赖专用工程系统，而不是通用 DeerFlow subagent 的自由协作。

### 4.3 哪些“看起来像 agent”的东西不属于本项目中的 DeerFlow subagent

`directionai_pptagent` 内部如果存在 orchestrator、phase skill、repair runtime 或其他 agent-like 组件，它们属于 PPT 专用子系统自身的 runtime 设计。

它们不等价于：

- 本项目中由 DeerFlow Lead Agent 动态孵化的新 subagent

这个边界必须写清楚，避免后续实现时混淆两个层次。

## 5. 系统分层

### 5.1 编排层

由 DeerFlow / DirectionAI workflow 负责：

- 判断是否需要生成 PPT
- 组织 lesson / exam / document 上下文
- 生成标准 `PPTTaskSpec`

这里允许的 DeerFlow 动态行为只有：

- 是否触发 PPT 生成
- 是否把 lesson artifact、exam artifact、document context 纳入本次请求
- 是否在失败后重试调用同一个 PPT 子系统入口

### 5.2 子系统适配层

由 `ppt_subsystem_adapter.py` 负责：

- 把 `PPTTaskSpec` 转成 `directionai_pptagent` 可执行请求
- 调起专用 PPT runtime
- 接收 preview / done / trace

### 5.3 兼容层

由 router + mapper 负责：

- 兼容旧前端请求字段
- 兼容旧前端结果字段
- 兼容旧前端 SSE 事件

## 6. 推荐工作流

```mermaid
flowchart TD
    A["Legacy / New Request"] --> B["Normalize To PPTTaskSpec"]
    B --> C["Build PPT Context From Lesson / Exam / Documents"]
    C --> D["PPTSubsystemAdapter"]
    D --> E["Outline / Research / Assets / Layout / Render / QA / Repair"]
    E --> F["Preview Stream Mapping"]
    E --> G["Done Result Mapping"]
    F --> H["Emit preview SSE"]
    G --> I["Emit done SSE"]
```

## 7. 不同于 lesson / exam 的地方

### 7.1 PPT 不建议拆成大量业务 Agent 自由发挥

lesson / exam 可以做原生多 Agent 化。  
PPT 不建议照这个思路走。

### 7.2 PPT 更像专用引擎

PPT 的核心难点在于：

- layout
- render
- preview
- syntax repair
- content QA
- visual QA
- benchmark / promotion / learned memory

这些都更像专用工程系统，而不是通用 Prompt 问题。

### 7.3 明确禁止的做法

下面这些都不应该出现：

- 让 DeerFlow 主 Agent 每次现造新的页面生成 subagent
- 让 DeerFlow 主 Agent 直接调底层 layout/render 工具自由排版
- 用“统一 Agent 架构”为理由，绕过 `directionai_pptagent` 的 QA、repair、trace 机制
- 把 PPT 核心逻辑重新退化成一串 prompt 和自由文本中间态

## 8. 应优先迁入的 `directionai_pptagent` 能力

应优先保留：

- `api.py` 中的兼容请求规范
- `backend/harness/agents/orchestrator`
- `backend/harness/runtime/*`
- `backend/models/schemas.py`
- `backend/tools/*`
- trace / preview / repair / benchmark 相关逻辑

## 9. 当前兼容字段映射

来自 `directionai_pptagent/api.py` 的重要兼容字段包括：

- `language -> output_language`
- `audience -> target_audience`
- `slides -> min_slides/max_slides`
- `page_limit -> min_slides/max_slides`
- `use_rag -> enable_web_search`
- `course/units/lessons/knowledge_points/constraint -> topic derivation`
- `document_text/document_name`

这套兼容逻辑建议复用，而不是重新发明。

## 10. Artifact 设计

推荐至少定义：

- `PPTTaskSpec`
- `PPTGenerationRequestArtifact`
- `PPTGenerationResultArtifact`
- `PPTTraceArtifact`

## 11. Tool 边界

PPT 相关工具应至少包括：

- 文档摘要 / 文档提取
- outline 规划
- asset 检索 / 生图
- layout / render
- preview 生成
- QA
- repair
- trace / benchmark

但这些工具不应全部暴露给通用 Lead Agent 直接自由调用。

### 11.1 正确的暴露方式

推荐只向 DirectionAI 编排层暴露少量高层入口，例如：

- `build_ppt_request_from_context`
- `run_ppt_subsystem`
- `map_ppt_preview_event`
- `map_ppt_done_result`

不推荐暴露给通用 Lead Agent 的底层能力包括：

- 原始 layout tool
- 原始 render tool
- 原始 repair tool
- 原始 preview image tool

## 12. Compatibility API 设计

必须兼容的行为：

- `POST /stream_ppt`
- `POST /generate_ppt`
- `POST /upload_document`
- `POST /evaluate/ppt`
- `GET /download_ppt/{filename}`
- `GET /preview_ppt/{filename}/{image_name}`

是否对外沿用相同路径，可以通过旧仓的 Nginx / BFF 层继续做映射。

## 13. 测试设计

至少要有：

- `backend/tests/contracts/ppt_*`
- `backend/tests/integration/ppt_*`
- `backend/tests/regression/ppt_*`

重点覆盖：

- request compatibility normalization
- preview event payload
- done payload
- download / preview url
- no-preview fallback
- upload document path
- trace presence
- 不存在 DeerFlow 动态孵化页面 subagent 的实现回归

## 14. 实现边界总结

PPT 链在本项目中的最终边界应当是：

- DeerFlow 负责编排
- DirectionAI 负责上下文整理和兼容层
- `directionai_pptagent` 负责实际 PPT runtime

因此，后续任何人看文档时都应直接按这个判断：

- PPT 链没有使用 DeerFlow 动态生成 subagent 来完成核心生成
- PPT 链也不允许把每次请求都变成一套新的页面角色组合
