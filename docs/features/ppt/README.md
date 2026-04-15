# PPT 功能文档入口

这个目录专门描述“PPT 生成链”的重构要求。

这条链与教案、试卷最不一样的地方是：

- 它不是纯文本生成链
- 它有强视觉质量要求
- 它已经有成熟的专用子系统基线
- 它必须保留 preview / render / QA / repair / trace

## 文档列表

### [requirements.md](./requirements.md)

定义 PPT 链应该做什么，以及哪些用户可见行为必须保留。

### [technical-design.md](./technical-design.md)

定义 PPT 链在当前仓库中的定位、与 DeerFlow 的边界，以及迁移 `directionai_pptagent` 的技术方案。

### [contracts.md](./contracts.md)

定义 PPT 的 request / preview / done / upload / trace / download 契约。

### [implementation-checklist.md](./implementation-checklist.md)

定义 PPT 子系统整合的实施顺序、质量门槛和回归要求。

## 一句话定位

PPT 链不是一个简单的“prompt 输出 markdown”问题，而是一个带渲染、预览、修复和质量闭环的专用子系统接入问题。
