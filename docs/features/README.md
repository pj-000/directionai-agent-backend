# 功能子文档入口

这里放的是面向重构实施的功能级子文档。

主文档解决的是：

- 总体目标
- 总体架构
- 总体工程规范
- 总体 bug 处理规则

这里的功能子文档解决的是：

- 教案、试卷、PPT 每一条链具体应该怎么重构
- 每条链应该拆哪些模块
- 每条链的 request / response / SSE / artifact / tool / skill 应该长什么样
- 每条链对 DeerFlow subagent 的使用边界是什么
- 每条链的迁移来源、测试要求和完成标准是什么

## 阅读顺序

建议按下面顺序阅读：

1. `docs/requirements.md`
2. `docs/architecture.md`
3. `docs/engineering.md`
4. 对应功能目录下的子文档

## DeerFlow Sub-Agent 使用矩阵

这部分是后续重构时最容易做偏的地方，所以单独固定下来。

| 功能 | 是否使用 DeerFlow 生成 subagent | 每次请求是否生成新的角色定义 | 运行时到底动态什么 | 不允许的错误做法 |
| --- | --- | --- | --- | --- |
| 教案 | 使用 | 否 | 是否启用 `KnowledgeGrounder`、`ActivityDesigner`、`LessonReviewer` 之类的固定模板实例 | 让主 Agent 每次根据课程主题现编一套新的 lesson 专家角色 |
| 试卷 | 使用 | 否 | 按题型和数量 fan-out 多个固定模板实例，并决定是否启用某类题型 worker | 让主 Agent 每次现造新的题型专家或 checker 体系 |
| PPT | 不把 DeerFlow subagent 用作核心生成能力 | 否 | 只动态决定是否生成 PPT、收集哪些上下文、何时调用 PPT 子系统 | 让 DeerFlow 主 Agent 自由孵化 PPT 页面写手、排版师、视觉修复师来替代专用子系统 |

这张表是全项目统一约束。

如果后续代码实现和这里冲突，优先按这里纠偏。

## 功能目录

### [lesson](./lesson/README.md)

教案链完整重构文档。

包含：

- 业务需求
- 技术设计
- 契约定义
- 实施清单
- DeerFlow subagent 使用方式和限制

### [exam](./exam/README.md)

试卷链完整重构文档。

包含：

- 题型蓝图
- fan-out 设计
- 输出契约
- 回归与验收
- DeerFlow subagent 的固定模板和实例化策略

### [ppt](./ppt/README.md)

PPT 链完整重构文档。

包含：

- PPT 专用子系统边界
- 与 DeerFlow 的衔接方式
- preview / done / trace / upload 契约
- 质量和迁移要求
- 为什么这条链不使用 DeerFlow 动态孵化 subagent 作为核心能力

## 使用原则

- 每条链的行为变化，先改对应功能目录里的 `requirements.md`
- 每条链的技术边界变化，先改对应功能目录里的 `technical-design.md`
- 每条链的接口、artifact、状态或 SSE 变化，先改对应功能目录里的 `contracts.md`
- 每条链的实施步骤、测试和收尾要求变化，先改对应功能目录里的 `implementation-checklist.md`
