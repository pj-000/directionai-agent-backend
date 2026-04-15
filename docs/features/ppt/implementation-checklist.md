# PPT 功能实施清单

## 1. 迁移来源

优先阅读：

- `/Users/sss/directionai/directionai_pptagent/api.py`
- `/Users/sss/directionai/directionai_pptagent/backend/harness/agents/orchestrator.py`
- `/Users/sss/directionai/directionai_pptagent/backend/harness/runtime/`
- `/Users/sss/directionai/directionai_pptagent/backend/models/schemas.py`
- `/Users/sss/directionai/directionai_pptagent/backend/tools/`

## 2. 第一阶段：接入 request / result schema

先补：

- `ppt_schemas.py`
- `ppt_request_builder.py`
- `ppt_result_mapper.py`

完成标准：

- request compatibility 明确
- preview / done / upload 契约明确

## 3. 第二阶段：PPT 子系统适配器

实现：

- `ppt_subsystem_adapter.py`
- `ppt_trace_service.py`

完成标准：

- 能调起成熟 PPT 子系统
- 能拿到 preview / done / trace
- DeerFlow 侧不存在页面级 subagent 自由孵化逻辑

## 4. 第三阶段：兼容路由

实现：

- `ppt_router.py`
- `compatibility_router.py`

完成标准：

- 对旧前端可继续暴露 `/stream_ppt` 等能力
- preview / done 字段兼容

## 5. 第四阶段：上传文档链路

实现：

- `upload_document` 接口
- 文档摘要 / 文本抽取
- 和 PPT request builder 的打通

## 6. 第五阶段：测试

必须补：

- contract tests
- integration tests
- regression tests

重点用例：

- topic 为空但可由 course/units/lessons 推导
- `slides` / `page_limit` 兼容
- preview 事件流
- no-preview fallback
- done 字段完整
- upload_document -> stream_ppt
- trace 可用

## 7. 第六阶段：质量回归

至少要做：

- 与 `directionai_pptagent` 基线样例对比
- 检查导出可打开
- 检查 preview 是否可用
- 检查视觉质量是否明显退化
- 检查实现中没有把核心 PPT 生成退化为 DeerFlow 通用 subagent 流

## 8. DoD

- PPT 兼容接口可运行
- preview / done 契约稳定
- 与 `directionai_pptagent` 基线相比无明显退化
- QA / repair / trace 路径保留
- 旧前端可继续接入
