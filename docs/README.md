# AI 后端重构文档入口

这套文档现在已经服务于当前仓库：

`directionai-agent-backend`

这个仓库已经以 DeerFlow 2.0 作为基底引入，并将作为 DirectionAI 新 AI 后端的正式重构基线。

后续无论是：

- 迁移 `DirectionAICloud` 里的旧 AI 能力
- 对接现有前端
- 把教案、试卷、PPT 重新组织成新的后端
- 修 bug 时判断该先改文档还是先改代码

都应优先以这 4 份主文档为准，而不是回到旧实现里重新猜业务边界。

## 先给结论

当前推荐方案已经落地为这个仓库本身，具体是：

1. 当前仓库 `directionai-agent-backend` 保持自己的 Git 历史
2. DeerFlow 源码作为基底导入当前仓库，而不是嵌套成子目录继续开发
3. 教案和试卷重构为新的多 Agent 工作流
4. PPT 不在 DeerFlow 内重新自由生成，而是迁入并复用 `/Users/sss/directionai/directionai_pptagent` 的成熟子系统能力
5. 通过 Compatibility API 对接 `/Users/sss/directionai/DirectionAICloud` 里的现有前端和旧主系统

这个决策的原因是：

- 当前仓库模块太多，包含前端、Java 主后端、Go 用户服务、Remotion、Nginx、监控、数据库初始化等，AI 后端重构会被非 AI 模块噪声干扰
- 这次重构不仅是代码迁移，而是一次架构范式升级，从“手写流程 + Prompt 拼接”转向“agent orchestration + skill/tool contracts + artifact first”
- 需要保留现有前端行为稳定。如果直接在当前仓库重构，很容易把联调层、兼容层和核心 AI 逻辑搅在一起
- PPT 子系统已经在外部独立仓库形成了更成熟的 harness 结构，适合直接作为这个新仓库里的专用模块基线

## 文档列表

### [claude-review-context.md](./claude-review-context.md)

这是一份给 Claude 或其他后续参与者看的“单文档交接摘要”。

适合在以下场景先读：

- 希望快速理解这次对话里最终确定的重构目标
- 希望把项目背景、重构路线、前端兼容要求、仓库策略一次性交给另一个模型
- 希望让别人先看一份总文档，再进入 4 份主文档

### [features/README.md](./features/README.md)

这是三大功能链的子文档入口。

适合在以下场景先读：

- 你已经理解总体架构，准备真正开始拆某一条链
- 你希望把教案、试卷、PPT 分开实施
- 你希望直接按照领域文档落代码

当前包含：

- [features/lesson](./features/lesson/README.md)
- [features/exam](./features/exam/README.md)
- [features/ppt](./features/ppt/README.md)

### [requirements.md](./requirements.md)

回答“系统应该做什么”。

重点包含：

- 新 AI 后端的目标和范围
- 前端哪些行为必须保持不变
- 教案、试卷、PPT 三类能力的输入、输出和质量要求
- 组合任务的行为要求
- 重构完成后的验收标准
- 当前前端正在使用的关键 SSE / 接口契约

如果你遇到的问题是：

- 用户到底应该看到什么结果
- 某个流程是不是应该允许
- 某个字段对前端意味着什么
- 修复是否会改变产品行为

先看这份。

### [architecture.md](./architecture.md)

回答“系统应该怎么拆，模块怎么协作”。

重点包含：

- 为什么当前仓库以 DeerFlow 为基底而不是直接在旧 monorepo 上大改
- 当前代码资产如何映射到这个新仓库
- 教案、试卷、PPT 三条链如何重构
- Compatibility API、Agent、Tool、Skill、Artifact、Storage 的边界
- 推荐的目录结构、依赖方向、运行方式和迁移顺序
- DeerFlow 上游代码和 DirectionAI 自有代码的边界

如果你遇到的问题是：

- 某个模块应不应该依赖另一个模块
- DeerFlow 应该负责什么，不应该负责什么
- PPT 到底是 Skill、Tool，还是独立子系统
- schema、状态机、artifact 应该怎么定

先看这份。

### [engineering.md](./engineering.md)

回答“代码应该怎么写，目录应该怎么组织，测试应该怎么补”。

重点包含：

- 项目结构规范
- 分层依赖规则
- schema、tool、skill、prompt 的代码规范
- 日志、trace、错误码、超时、重试规范
- 测试分层与回归测试要求
- 开发流程和提交流程

如果你遇到的问题是：

- 一个模块应该放在哪个目录
- 怎么命名 schema / service / tool
- 什么时候必须先补测试
- prompt、skill、tool 的文件应该怎么放

先看这份。

### [bug-policy.md](./bug-policy.md)

回答“出 bug 以后应该先改文档还是先改代码”。

重点包含：

- bug 四分类
- 每类问题的典型信号
- 每类问题的处理顺序
- 什么时候先改 `requirements.md`
- 什么时候先改 `architecture.md`
- 什么时候直接补测试后修代码

如果你对修复顺序拿不准，先看这份。

## 推荐使用方式

### 在正式开始重构前

按顺序阅读：

1. `requirements.md`
2. `architecture.md`
3. `engineering.md`
4. `bug-policy.md`

### 在新需求或行为调整出现时

按顺序处理：

1. 先改 `requirements.md`
2. 如涉及边界或契约，再改 `architecture.md`
3. 再实现代码
4. 最后更新 `engineering.md` 中相关开发规范或测试要求

### 在疑似架构问题出现时

按顺序处理：

1. 先看 `architecture.md`
2. 判断是否需要调整边界、目录、依赖、状态机、schema
3. 再改代码

### 在纯实现 bug 出现时

按顺序处理：

1. 先看 `bug-policy.md`
2. 若确认规范没问题，按 `engineering.md` 要求先补测试
3. 再修代码

## 文档维护原则

- 不再新增大量零散文档
- 需求和行为定义只写进 `requirements.md`
- 架构边界和目录结构只写进 `architecture.md`
- 代码规范、测试规范和开发流程只写进 `engineering.md`
- bug 分类和处理顺序只写进 `bug-policy.md`

如果后续发现某个问题需要额外说明，优先把内容并入这 4 份主文档，而不是再平行创建一堆新文件。

## 当前仓库与旧仓关系

### 当前仓库负责

- DeerFlow 基底
- 新 AI 后端主实现
- 教案、试卷、PPT 的新编排
- Compatibility API
- 新的工程规范和测试

### 旧仓 `/Users/sss/directionai/DirectionAICloud` 继续负责

- 前端页面
- Java 主后端
- 现有反向代理和联调环境
- 过渡期接入

### 外部仓 `/Users/sss/directionai/directionai_pptagent` 负责

- 作为 PPT 专用子系统的成熟基线

## 执行原则

后续重构尽量按这个顺序推进：

1. 先冻结需求和前端兼容边界
2. 再明确技术边界和目录规划
3. 再落 Compatibility API
4. 再迁教案链和试卷链
5. 最后把 PPT 专用子系统整合进来并做效果回归
