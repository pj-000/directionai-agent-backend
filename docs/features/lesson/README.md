# 教案功能文档入口

这个目录专门描述“教案生成链”的重构要求。

后续如果有人只负责教案链，优先阅读这里，而不是从旧代码里直接猜流程。

## 文档列表

### [requirements.md](./requirements.md)

定义教案链应该做什么。

适合在以下情况先看：

- 你不确定教案链最终应该输出什么
- 你不确定用户可见行为是什么
- 你不确定哪些输入字段必须支持

### [technical-design.md](./technical-design.md)

定义教案链应该怎么拆、怎么实现。

适合在以下情况先看：

- 你要开始写代码
- 你要决定 lesson 模块怎么拆
- 你要决定 Agent / Tool / Artifact 边界

### [contracts.md](./contracts.md)

定义教案链的 request / response / SSE / artifact / tool / skill 契约。

适合在以下情况先看：

- 你要做 Compatibility API
- 你要补 schema
- 你要补 contract test

### [implementation-checklist.md](./implementation-checklist.md)

定义教案链的实施顺序、测试要求和完成标准。

适合在以下情况先看：

- 你要实际排重构任务
- 你要评估这条链还缺什么
- 你要做收尾验收

## Sub-Agent 结论

教案链是一个“可原生多 Agent 化”的链路。

这里的“多 Agent 化”有明确约束：

- 使用 DeerFlow 的 subagent 能力
- 只使用固定角色模板
- 每次请求不会重新发明一套全新的角色定义
- 运行时只决定这次是否启用哪些固定角色、是否需要局部回炉

它应该：

- 充分利用教材、RAG 和用户约束
- 输出稳定的 lesson artifact
- 为试卷和 PPT 提供高质量上游上下文

它不应该继续绑定旧的硬编码 workflow JSON。
