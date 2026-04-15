# DirectionAI AI 后端重构交接文档

## 1. 这份文档的用途

这份文档是给后续继续参与项目的人看的，尤其适合直接给 Claude 或其他代码助手阅读。

它的目标不是替代主文档，而是把这次对话中已经明确下来的事情一次性讲清楚，包括：

- 我到底想实现什么效果
- 为什么要这么做
- 当前代码和仓库现状是什么
- 我最终选定的技术路线是什么
- 哪些东西不能动
- 哪些东西可以彻底重构
- 哪些文档是后续实现时必须遵循的
- 现在这个新仓库已经做到了哪一步

如果后续要继续补文档、改架构、写代码、迁移旧逻辑，建议先读完这份文档，再去读主文档。

## 2. 当前项目背景

我在做的是一个 AI 教育系统。

当前旧系统里，前端是结构化表单输入，用户会填写类似这些字段：

- 课程名称
- 课时 / 单元 / 章节
- 字数限制
- 页数限制
- 附加要求
- 知识点
- 题型分布
- 难度分布

旧后端主要采用：

- FastAPI 单体
- 手写 workflow
- Prompt 串联
- 正则 / 文本抽取
- RAG / Faiss
- 自己写的 PPT 排版与渲染逻辑

旧代码主要分布在：

- 旧 monorepo：
  - `/Users/sss/directionai/DirectionAICloud`
- 其中和 AI 后端重构相关的主要目录：
  - `evoagentx/`
  - `python-backend/`
  - 仓内 `pptagent/`
- 外部成熟 PPT 子系统基线：
  - `/Users/sss/directionai/directionai_pptagent`

## 3. 我真正想达到的效果

我的最终目标不是“简单把旧代码搬过来”，而是要做一次真正的 AI 后端重构。

目标包括：

1. 保持旧前端页面和交互尽量不变
2. 后端内部可以彻底重构
3. 用 DeerFlow 2.0 作为新的 agent orchestration 基底
4. 教案和试卷采用新的多 Agent 工作流
5. PPT 不做成自由发挥的通用 Agent，而是作为专用子系统接入
6. 后续修 bug 时，团队能先根据文档判断：
   - 先改需求文档
   - 先改技术文档
   - 先补测试
   - 还是直接改代码
7. 文档要足够详细，让别人只看文档也能理解项目并开始重构

换句话说，我要的不是：

- 临时补丁式改造
- 在旧 monorepo 里继续堆逻辑
- 把所有东西都强行抽象成一个通用 Agent

我要的是：

- 一个清晰分层、可长期维护、可迁移旧系统能力、对前端兼容、对团队友好的新 AI 后端仓库

## 4. 当前最终确定的仓库策略

### 4.1 结论

最终决定是：

- 新建独立仓库：
  - `directionai-agent-backend`
- 不直接在旧的 `DirectionAICloud` monorepo 里大改主 AI 后端

### 4.2 原因

旧仓 `DirectionAICloud` 过于复杂，同时包含：

- 前端
- Java 主后端
- Go 用户服务
- Nginx
- Remotion
- 监控
- 数据库初始化
- 多种部署与联调逻辑

如果直接在旧仓里大改 AI 后端，会带来这些问题：

1. AI 重构与非 AI 模块耦合过重
2. 很难区分“兼容旧前端的适配层”和“新 AI 核心引擎”
3. 很难沉淀一套清晰的 AI 工程规范和目录结构
4. 很容易继续把新旧逻辑混在一起
5. PPT 已经在外部有更成熟的专用子系统，不适合再被拉回旧形态

### 4.3 当前仓库状态

新的远端仓库已经建立：

- [directionai-agent-backend](https://github.com/pj-000/directionai-agent-backend)

本地路径：

- `/Users/sss/directionai/directionai-agent-backend`

## 5. DeerFlow 的使用方式

### 5.1 不是“引用”，而是“基底导入”

当前新仓库已经把 DeerFlow 2.0 作为基底源码导入进来了。

采用的方式不是：

- 把 DeerFlow 作为 git submodule
- 或者把 DeerFlow 放到一个子目录里继续套娃开发

而是：

- 把 DeerFlow 最新源码拷到当前仓库里
- 保留当前仓库自己的 Git 历史和远端
- 后续在这个基底上继续做 DirectionAI 的业务重构

### 5.2 当前导入的 DeerFlow 基线

导入时使用的 DeerFlow 上游 HEAD 是：

- `8760937439e2722203f7d759414b667f20bbb285`

### 5.3 DeerFlow 在这里负责什么

DeerFlow 在这个项目里主要负责：

- lead / sub-agent 编排能力
- sub-agent 运行机制
- memory / runtime / sandbox / tracing / skills / tools 这类通用 harness 能力

DeerFlow 不应直接负责：

- DirectionAI 的教学业务规则
- 旧前端兼容层
- PPT 的专用排版与渲染细节

## 6. 三条核心业务链的最终技术路线

## 6.1 教案生成

教案生成允许彻底摆脱旧 `evoagentx` 的固定 workflow JSON。

新的方向是：

- 使用 DeerFlow 风格的多 Agent 结构
- 固定角色模板
- 运行时动态实例化
- 输出稳定 artifact，而不是只返回一大段自由文本

建议角色包括：

- `CurriculumPlanner`
- `KnowledgeGrounder`
- `PedagogyDesigner`
- `ActivityDesigner`
- `LessonReviewer`

要求：

- 能利用教材 / RAG / 上传文档
- 结构完整
- 能作为试卷和 PPT 的上游上下文
- 最终对前端兼容输出 `lesson_plan`

## 6.2 试卷生成

试卷生成也允许彻底摆脱旧 `evoagentx/exam_generator.py` 的固定流程。

新的方向是：

- 用 DeerFlow 做 fan-out 并行
- blueprint + 各题型 generator + checker + assembler

建议角色包括：

- `ExamBlueprintPlanner`
- `SingleChoiceGenerator`
- `MultipleChoiceGenerator`
- `TrueFalseGenerator`
- `FillBlankGenerator`
- `ShortAnswerGenerator`
- `ProgrammingGenerator`
- `AnswerAnalysisChecker`
- `DedupReviewer`
- `PaperAssembler`

要求：

- 按题型和数量配置生成
- 输出结构稳定
- 支持后续校验与回归测试
- 最终对前端兼容 `question/questions/output.question`

## 6.3 PPT 生成

PPT 生成是最特殊的一条链。

这里最终确定的原则是：

- 不把 PPT 重写成 DeerFlow 内部自由发挥的通用 Agent 创作系统
- DeerFlow 主要负责任务编排和上下文整理
- 真正的 PPT 生成、render、preview、QA、repair、trace 交给专用子系统

主基线来源是：

- `/Users/sss/directionai/directionai_pptagent`

必须保留的能力包括：

- phase-based skills
- orchestrator / pipeline
- runtime memory
- repair orchestrator
- harness trace
- benchmark / promotion
- preview / output artifacts

结论就是：

- 教案和试卷是 DeerFlow 原生多 Agent 重构对象
- PPT 是 DeerFlow 编排下的专用子系统，不是被 DeerFlow 完全吞掉的通用技能块

## 7. 前端兼容要求

这是整个重构里最重要的硬约束之一。

### 7.1 前端页面尽量不变

默认冻结的内容包括：

- 页面结构
- 表单输入方式
- 结果展示方式
- 下载与预览交互
- 流式展示方式

### 7.2 当前前端已经在用的关键入口

根据旧仓前端代码，当前至少已经依赖这些入口语义：

- `/evoapi/upload_document`
- `/evoapi/stream_lesson_plan`
- `/evoapi/stream_exam`
- `/pptagentapi/stream_ppt`
- `/pptagentapi/stream_evaluate/ppt`

新的 AI 后端需要通过 Compatibility API 维持这些语义。

### 7.3 当前前端已经依赖的 SSE 事件

当前前端 SSE 客户端依赖这些事件类型：

- `thinking_start`
- `thinking_chunk`
- `thinking_end`
- `progress`
- `preview`
- `done`
- `error`

Compatibility API 必须继续输出这些事件语义。

### 7.4 当前前端已经消费的关键完成态字段

#### 教案

- `lesson_plan`

#### 试卷

- `question`
- `questions`
- `output.question`

#### PPT

- `markdown_content`
- `display_url`
- `download_url`
- `preview_images`
- `preview_warning`
- `biz_id`

这意味着：

- 后端内部可以完全重写
- 但兼容层最终必须把内部结果映射回这些前端已依赖的字段

## 8. 当前仓库的代码边界

当前仓库已经基于 DeerFlow 实际目录展开，因此后续不要再凭空设计一个和现状脱节的新目录树。

### 8.1 DeerFlow 上游核心区

路径：

- `backend/packages/harness/deerflow/`

规则：

- 视为上游核心区
- 尽量少改
- 如果必须改，要明确知道自己在改上游核心
- 优先通过包装、注册、扩展、外部 skill / tool / router 的方式接入

### 8.2 DirectionAI 自有业务区

路径：

- `backend/packages/directionai/`

这里是后续主开发区。

教案、试卷、PPT 的领域逻辑主要都应放在这里，包括：

- compat
- lesson
- exam
- ppt
- schemas
- tools
- storage
- telemetry
- workflows

### 8.3 Compatibility API 区

路径：

- `backend/app/gateway/routers/`

职责：

- HTTP 路由
- SSE 路由
- 旧前端字段映射
- 旧前端结果字段映射

不该放：

- 复杂业务生成逻辑
- 大量 Prompt 逻辑
- PPT render 细节

## 9. 项目目录树在哪个文档里

完整项目目录树统一放在：

- [docs/architecture.md](./architecture.md)

重点章节：

- `5.1 完整项目目录树`

那里现在已经明确写了：

- 根目录结构
- DeerFlow 上游区
- DirectionAI 域逻辑区
- tests 结构
- skills 结构
- examples 结构
- 建议补齐的最小实现文件集合

工程文档 `engineering.md` 则更偏：

- 每个目录该放什么
- 不该放什么
- 分层依赖怎么控制
- 测试和代码规范怎么执行

## 10. 我要求的主文档体系

我不想维护很多零散文档。

我只想维护少量主文档，但这些主文档必须足够详细，能直接指导重构。

当前主文档是：

- [docs/README.md](./README.md)
- [docs/requirements.md](./requirements.md)
- [docs/architecture.md](./architecture.md)
- [docs/engineering.md](./engineering.md)
- [docs/bug-policy.md](./bug-policy.md)

它们的职责如下：

### `requirements.md`

回答：

- 系统应该做什么
- 前端哪些行为不能变
- 教案 / 试卷 / PPT / 组合任务目标是什么
- 验收标准是什么

### `architecture.md`

回答：

- 系统怎么拆
- 仓库和旧系统怎么分工
- DeerFlow 和 DirectionAI 的边界怎么定
- 教案 / 试卷 / PPT 的技术方案是什么
- Compatibility API 怎么定位
- 完整项目目录树是什么
- 旧代码怎么迁移

### `engineering.md`

回答：

- 代码应该怎么组织
- 哪层可以依赖哪层
- schema / tool / skill / SSE contract 怎么定
- 测试怎么分层
- 代码评审重点看什么
- 开发流程怎么执行

### `bug-policy.md`

回答：

- 遇到 bug 时是先改需求文档
- 先改技术文档
- 先补测试
- 还是直接改代码

## 11. 我对 bug 修复流程的要求

以后遇到 bug，不希望团队默认直接改代码。

而是先分类。

### 11.1 第一类：需求 / 设计层问题

如果“正确行为到底是什么”都还不清楚，或者修复会改变用户可见行为：

- 先改 `requirements.md`
- 再改代码

### 11.2 第二类：架构 / 契约层问题

如果问题本质上是：

- 模块边界
- 依赖方向
- schema
- 状态机
- SSE 契约
- tool / skill contract

那么：

- 先改 `architecture.md`
- 必要时同步改 `engineering.md`
- 先补 contract / integration tests
- 再改代码

### 11.3 第三类：纯实现问题

如果规范没错，只是代码没做到：

- 先补失败测试 / 回归测试
- 再改代码

### 11.4 第四类：纯重构

如果不改外部行为：

- 可以直接改代码
- 但必须靠测试兜底

## 12. 代码规范和测试规范有没有写

有，而且已经落在主文档里。

主要在：

- [docs/engineering.md](./engineering.md)

里面已经写了：

- 项目结构规范
- 分层依赖规则
- 命名规范
- schema / contract 规范
- prompt / skill 规范
- 日志、trace、异常规范
- 测试规范
- 开发流程规范
- 代码评审关注点
- PPT / Compatibility API / DeerFlow 核心区的高风险提示

## 13. 当前仓库已经完成了什么

截至目前，已经完成的事情包括：

1. 新仓库已经建立并可访问
2. DeerFlow 2.0 最新源码已经导入作为基底
3. 根 README 已经改成 DirectionAI 版本入口
4. 主文档已经迁入并重写成当前仓库语境
5. 文档中已经写明：
   - 前端兼容要求
   - 三条主链的技术路线
   - 仓库目录树
   - 代码规范
   - bug 处理规则
6. DirectionAI 域包骨架已经建立，包括：
   - `backend/packages/directionai/compat`
   - `backend/packages/directionai/lesson`
   - `backend/packages/directionai/exam`
   - `backend/packages/directionai/ppt`
   - `backend/packages/directionai/schemas`
   - `backend/packages/directionai/tools`
   - `backend/packages/directionai/storage`
   - `backend/packages/directionai/telemetry`
   - `backend/packages/directionai/workflows`
7. 测试目录骨架已经建立：
   - `backend/tests/contracts`
   - `backend/tests/integration`
   - `backend/tests/regression`
   - `backend/tests/directionai`
8. 这些改动已经推送到远端 `main`

## 14. 当前远端状态

远端仓库：

- [directionai-agent-backend](https://github.com/pj-000/directionai-agent-backend)

当前已推送的关键提交包括：

- `573834d`
  `Bootstrap DirectionAI agent backend with DeerFlow base`
- `f1f6da4`
  `Expand architecture docs with detailed project tree`

## 15. 现在还没有做的事情

虽然基底和文档已经搭好了，但以下内容还没有真正开始写实现：

- Compatibility API 真实路由实现
- lesson 链真实 workflow
- exam 链真实 workflow
- PPT 子系统正式迁入
- 旧前端与新仓库的联调
- 真实 contract tests / integration tests / regression tests
- 旧代码的实际迁移

所以当前状态更准确地说是：

- 架构基线和文档基线已经建立完成
- 代码重构工作可以正式开始

## 16. 希望 Claude 接下来重点帮我看什么

如果把这份文档交给 Claude，我希望它重点检查：

1. 这套文档是否已经足够详细，能不能支撑别人开始重构
2. 哪些地方仍然太抽象
3. 哪些地方需要补成更明确的 contract 或模板
4. 完整目录树是否合理，是否和当前 DeerFlow 基底结构匹配
5. DeerFlow 上游区与 DirectionAI 自有区的边界是否足够清楚
6. Compatibility API 的定义是否已经足够具体
7. 教案、试卷、PPT 三条链的拆分是否合理
8. 代码规范、测试规范、bug-policy 是否足够可执行
9. 还缺哪些“必须先写清楚”的内容，才能真正做到“团队先看文档，再重构代码”

## 17. 一句话总结

这个项目现在真正想做的事情是：

在不大改现有前端体验的前提下，以 DeerFlow 为基底，重建一个新的 DirectionAI AI 后端仓库；教案和试卷走新的多 Agent 工作流，PPT 保持专用子系统路线；并用少量但足够详细的主文档，统一管理需求、架构、工程规范和 bug 修复顺序，让后续重构和维护都能按文档推进。
