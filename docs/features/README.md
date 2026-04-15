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
- 每条链的迁移来源、测试要求和完成标准是什么

## 阅读顺序

建议按下面顺序阅读：

1. `docs/requirements.md`
2. `docs/architecture.md`
3. `docs/engineering.md`
4. 对应功能目录下的子文档

## 功能目录

### [lesson](./lesson/README.md)

教案链完整重构文档。

包含：

- 业务需求
- 技术设计
- 契约定义
- 实施清单

### [exam](./exam/README.md)

试卷链完整重构文档。

包含：

- 题型蓝图
- fan-out 设计
- 输出契约
- 回归与验收

### [ppt](./ppt/README.md)

PPT 链完整重构文档。

包含：

- PPT 专用子系统边界
- 与 DeerFlow 的衔接方式
- preview / done / trace / upload 契约
- 质量和迁移要求

## 使用原则

- 每条链的行为变化，先改对应功能目录里的 `requirements.md`
- 每条链的技术边界变化，先改对应功能目录里的 `technical-design.md`
- 每条链的接口、artifact、状态或 SSE 变化，先改对应功能目录里的 `contracts.md`
- 每条链的实施步骤、测试和收尾要求变化，先改对应功能目录里的 `implementation-checklist.md`
