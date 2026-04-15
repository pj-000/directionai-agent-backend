# 试卷功能文档入口

这个目录专门描述“试卷生成链”的重构要求。

这条链和教案链不同，它最重要的特点是：

- fan-out 并行生成
- 题型配置驱动
- 输出结构必须稳定
- 下游通常不是继续生成，而是直接供前端展示、导出和评估

## 文档列表

### [requirements.md](./requirements.md)

定义试卷链应该做什么。

### [technical-design.md](./technical-design.md)

定义试卷链应该怎么拆、怎么做 fan-out、怎么聚合结果。

### [contracts.md](./contracts.md)

定义试卷链 request / response / SSE / artifact / tool 契约。

### [implementation-checklist.md](./implementation-checklist.md)

定义试卷链的实施顺序、测试要求和完成标准。

## Sub-Agent 结论

试卷链是一个最适合做“蓝图规划 + 题型并行生成 + checker + assembler”的链路。

它也是三条链里最适合使用 DeerFlow subagent 的地方，但要注意：

- subagent 模板是固定的，不是每次新造
- 动态的是实例数量和启用策略，不是角色定义本身
- 最常见的动态行为是按题型数量 fan-out 多个同模板实例
