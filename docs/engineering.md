# AI 后端工程规范

## 1. 文档定位

这份文档定义的是当前仓库 `directionai-agent-backend` 的工程规则和代码规范。

它的目标不是约束语法细节，而是确保后续重构时：

- 项目结构不会再次变得混乱
- 模块职责不会重新耦合
- Tool、Skill、Prompt、Schema 不会四处散落
- 测试能兜住重构风险
- 新同学进入项目时，能快速知道代码该写在哪、该怎么测、什么情况下该先补文档

如果后续修 bug 时发现：

- 规范本身没问题，只是代码没做到
- 不确定模块应该放在哪个目录
- 不确定某个能力应该写成 service 还是 tool
- 不确定测试应该补在哪一层

优先看这份文档。

## 2. 项目结构规范

当前仓库已经以 DeerFlow 的实际目录为基底，因此规范要以现有结构为准，而不是凭空再造一个目录树。

推荐的实现落点如下：

```text
directionai-agent-backend/
├─ docs/
├─ backend/
│  ├─ app/
│  │  └─ gateway/
│  │     └─ routers/
│  ├─ packages/
│  │  ├─ harness/
│  │  │  └─ deerflow/
│  │  └─ directionai/
│  │     ├─ compat/
│  │     ├─ lesson/
│  │     ├─ exam/
│  │     ├─ ppt/
│  │     ├─ schemas/
│  │     ├─ tools/
│  │     ├─ storage/
│  │     ├─ telemetry/
│  │     └─ workflows/
│  └─ tests/
├─ skills/
│  └─ directionai/
├─ scripts/
└─ examples/
```

### 2.1 `backend/app/gateway/routers/`

适合放：

- FastAPI 路由
- 请求校验
- SSE 输出
- Compatibility API 的字段映射

不适合放：

- 教案、试卷、PPT 的业务生成逻辑
- 模型调用细节
- 大段 Prompt 组装逻辑

### 2.2 `backend/packages/harness/deerflow/`

适合放：

- DeerFlow 上游通用 harness 能力
- 上游 runtime、memory、sandbox、subagent、skill runtime、tool runtime

不适合放：

- DirectionAI 的业务规则
- 针对旧前端的兼容补丁
- 教案、试卷、PPT 的领域特化逻辑

### 2.3 `backend/packages/directionai/`

适合放：

- 固定 Agent 模板定义
- 教案 / 试卷 / PPT 域逻辑
- schema
- workflow
- tool wrapper
- artifact mapping
- telemetry

不适合放：

- 文件系统操作
- 外部 API 适配代码
- 复杂业务 if/else

### 2.4 `backend/packages/directionai/compat/`

适合放：

- 旧前端字段到内部 `TaskSpec` 的映射
- 旧前端响应字段的回填
- SSE 事件 payload 组装

不适合放：

- 工具实现
- 复杂格式解析
- 大量 schema 兼容补丁

### 2.5 `backend/packages/directionai/lesson/`

适合放：

- 教案任务规格
- 教案子代理模板
- 教案 orchestration
- 教案 artifact 组装
- reviewer / repair 逻辑

### 2.6 `backend/packages/directionai/exam/`

适合放：

- 试卷蓝图
- 各题型生成器
- 去重与 checker
- paper assembler

### 2.7 `backend/packages/directionai/ppt/`

适合放：

- PPT 子系统接入层
- PPT request builder
- `directionai_pptagent` 迁入后的运行时封装
- preview / download / trace mapping

### 2.8 `backend/packages/directionai/schemas/`

适合放：

- Pydantic models
- Request / Response
- TaskSpec
- Artifact
- State
- ToolResult

### 2.9 `backend/packages/directionai/tools/`

适合放：

- RAG
- 文档解析
- 检索
- 评分 / 校验
- 领域工具包装

### 2.10 `backend/packages/directionai/storage/`

适合放：

- artifact 持久化
- trace store
- runtime memory 包装
- vector store 包装

### 2.11 `backend/packages/directionai/telemetry/`

适合放：

- structured logging
- run trace
- metrics

### 2.12 `skills/directionai/`

适合放：

- DirectionAI 自有的 skill 说明
- lesson / exam / ppt skill
- bug classify / doc update skill

### 2.13 `backend/tests/`

建议继续保留 DeerFlow 原有测试，同时新增：

- `backend/tests/contracts/`
- `backend/tests/integration/`
- `backend/tests/regression/`
- `backend/tests/directionai/`

## 3. 分层依赖规则

推荐固定依赖方向：

`gateway routers -> directionai domain -> tools/storage -> deerflow harness runtime`

并补充以下规则：

- `agents` 可以被 orchestrator 使用，但不应直接越层依赖底层 SDK
- `schemas` 可以被各层引用，但 schema 层不依赖业务实现
- `telemetry` 可以被各层调用，但不反向依赖业务层

### 3.1 禁止事项

- `api` 直接写业务生成逻辑
- `api` 直接调用底层模型 SDK
- `agents` 直接调用 OpenAI/Anthropic/DeerFlow 原始 client
- `orchestrators` 到处写 ad-hoc dict 兼容逻辑
- `services` 悄悄承担状态机职责
- `tools` 偷偷调用其他高层 service 来“图方便”
- `utils` 混入业务逻辑
- `backend/packages/directionai/*` 直接把上游 DeerFlow 内部实现 copy 一份出来魔改

## 4. 命名规范

### 4.1 文件与目录

- 目录名：小写下划线
- Python 文件名：小写下划线
- 不使用 `misc.py`、`helper.py`、`common.py` 这类无边界名字，除非内容真的通用而且低层

### 4.2 类、函数、变量

- 类名：PascalCase
- 函数名：snake_case
- 变量名：snake_case
- 常量：全大写下划线

### 4.3 Schema 命名

- 外部请求：`*Request`
- 内部任务规格：`*TaskSpec`
- 中间产物：`*Artifact`
- 最终返回：`*Result`
- 状态对象：`*State`
- 工具返回：`ToolResult`

### 4.4 模块命名建议

- lesson 相关：`lesson_*`
- exam 相关：`exam_*`
- ppt 相关：`ppt_*`
- compatibility 相关：`compat_*`
- artifact 相关：`*_artifact.py`
- schema 相关：`*_schemas.py`

### 4.5 Workflow 命名

- 教案：`lesson_plan_workflow`
- 试卷：`exam_generation_workflow`
- PPT：`ppt_dispatch_workflow`

不要使用含糊命名，如：

- `main_workflow`
- `new_workflow`
- `run_flow`

## 5. Schema 与 Contract 规范

### 5.1 Schema First

凡是跨层传递的数据，原则上先定义 schema，再写实现。

尤其是：

- API Request / Response
- TaskSpec
- Artifact
- ToolResult
- SSE Event Payload
- TaskState

### 5.2 ToolResult 统一格式

推荐统一结构：

```json
{
  "success": true,
  "data": {},
  "error": null,
  "meta": {}
}
```

其中：

- `success`：布尔值
- `data`：成功时的结构化结果
- `error`：失败时的错误对象
- `meta`：重试信息、耗时、版本、provider 等补充信息

### 5.3 Error 对象规范

推荐最少包含：

```json
{
  "code": "STRING_CODE",
  "message": "human readable message"
}
```

不要只返回一段自由文本。

### 5.4 Skill 输出规范

- Skill 输出尽量结构化
- 自由文本只允许出现在明确字段里，如 `content`、`reasoning_summary`、`markdown`
- 不能让下游依赖不稳定的自然语言格式去做正则抽取

### 5.5 SSE Contract 规范

兼容层输出 SSE 时，优先保持以下事件模型：

- `thinking_start`
- `thinking_chunk`
- `thinking_end`
- `progress`
- `preview`
- `done`
- `error`

如果内部 runtime 有更多细粒度事件，应先在兼容层聚合，再决定是否向前端透出。

## 6. Prompt 与 Skill 规范

### 6.1 Prompt 放置规则

- 运行时真正会被加载的 prompt，优先放在 `backend/packages/directionai/` 对应领域目录内
- Skill 说明文档或 skill 级模板，放在 `skills/directionai/` 或对应 skill 目录

### 6.2 Prompt 编写规则

- 明确角色
- 明确输入结构
- 明确输出结构
- 明确禁止事项
- 明确失败时应如何返回

不推荐：

- 超长自由发挥式 prompt
- 把业务规则、状态机、工具契约全部硬编码进一段不可维护的大 prompt

### 6.3 Skill 设计规则

- Skill 是可维护资产，不是一次性 prompt 碎片
- 每个 Skill 应有清楚的输入、输出、用途和边界
- Skill 不替代代码里的确定性逻辑

### 6.4 PPT 特殊规则

- PPT 的排版规则不应塞进通用 Lead Agent 的 prompt
- PPT 的页面骨架、渲染、评估、修复逻辑应继续留在 PPT 子系统内
- 上游 `directionai_pptagent` 的运行时经验不要在迁移时简化成只有一个 prompt 文件

## 7. 日志、Trace 与异常规范

### 7.1 标识要求

所有长任务至少带：

- `request_id`
- `run_id`
- `task_type`

如果是组合任务，还应能定位子任务。

### 7.2 日志要求

- 使用结构化日志
- 日志应带阶段信息
- 日志应能定位具体 tool、workflow、artifact

### 7.3 Trace 要求

至少应记录：

- 调用了哪些主要阶段
- 调用了哪些 tools
- 哪一步失败
- 哪些 artifact 被产出

PPT 应保留更细粒度 trace。

### 7.4 异常要求

- 对外错误必须有稳定 code
- 重试型错误与不可重试错误要区分
- 不允许吞异常后只返回 `failed`

## 8. 测试规范

推荐测试目录：

- `backend/tests/contracts/`
- `backend/tests/integration/`
- `backend/tests/regression/`
- `backend/tests/directionai/`
- 如有需要，保留上游 `backend/tests/*` 的原始 DeerFlow 测试

### 8.1 单元测试

覆盖：

- schema 校验
- 工具函数
- service 级规则函数
- 状态转换函数

### 8.2 集成测试

覆盖：

- lesson 链模块协作
- exam 链模块协作
- ppt dispatch 到子系统的协作
- RAG 与 artifact 流转

### 8.3 Contract Test

必须覆盖：

- Compatibility API 请求和响应结构
- SSE 事件结构
- ToolResult 结构
- 关键 Skill 输出结构
- lesson `done.lesson_plan`
- exam `done.question/questions`
- ppt `preview` 和 `done` 字段

### 8.4 Regression Test

以下情况必须补回归测试：

- 纯实现 bug 被修复后
- 复杂 bug 二次出现风险高
- PPT 修复涉及视觉或导出质量

### 8.5 E2E Test

至少覆盖：

- 只生成教案
- 只生成试卷
- 只生成 PPT
- 教案 + PPT + 试卷
- 旧前端直接接当前仓库 API 的联调

### 8.6 什么时候必须先补测试

- 纯实现 bug
- parser / validator 问题
- 重试逻辑问题
- 空值 / 边界值问题
- 已出现过一次的复杂 bug

## 9. 开发流程规范

标准开发顺序：

1. 先分类问题
2. 判断是否先改文档
3. 判断是否先补测试
4. 再改代码
5. 跑测试
6. 做联调验证

### 9.1 改行为时

- 先改 `requirements.md`
- 如涉及边界，再改 `architecture.md`
- 再改代码

### 9.2 改边界时

- 先改 `architecture.md`
- 再补 contract / integration tests
- 再改代码

### 9.3 纯实现 bug 时

- 先补失败测试
- 再修代码

## 10. 代码评审关注点

以后评审时优先看：

- 是否破坏分层
- 是否引入隐性耦合
- 是否让 schema 漂移
- 是否把业务逻辑塞进 `backend/app/gateway/routers/` 或其他不该承载业务的入口层
- 是否把 PPT 的专用规则错误地下沉到通用层
- 是否补了合适层级的测试
- 是否不必要地侵入修改了 `backend/packages/harness/deerflow/`

## 11. 文档维护规则

当前只维护 4 份主文档：

- `requirements.md`
- `architecture.md`
- `engineering.md`
- `bug-policy.md`

更新原则：

- 行为变化，改 `requirements.md`
- 架构、目录、边界、契约变化，改 `architecture.md`
- 工程规范、测试规范、编码规则变化，改 `engineering.md`
- bug 分类和修复顺序规则变化，改 `bug-policy.md`

## 12. 高风险区域特别说明

### 12.1 PPT

PPT 是高风险区域。

默认高风险修改包括：

- 改动 render/export 主链
- 改动 preview 生成逻辑
- 改动 QA / repair 流程
- 为了“统一架构”弱化视觉质量

### 12.2 Compatibility API

兼容层是高敏感区域。

任何影响前端字段语义、状态显示、SSE 结构的修改，都需要有 contract test。

### 12.3 任务状态机

状态机一旦乱掉，前端联调会变得很难排查。

任何状态语义变化，都必须谨慎对待。

### 12.4 上游 DeerFlow 核心

`backend/packages/harness/deerflow/` 属于高风险区域。

除非确定需要修上游核心问题，否则优先通过：

- 包装
- 注册
- 外部 skill
- 外部 tool
- DirectionAI 域逻辑包

来完成扩展。
