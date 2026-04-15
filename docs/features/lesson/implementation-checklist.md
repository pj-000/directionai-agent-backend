# 教案功能实施清单

## 1. 文档用途

这份文档不是讲原则，而是讲“实际落地时应该按什么顺序做”。

## 2. 迁移来源

优先阅读旧代码：

- `/Users/sss/directionai/DirectionAICloud/evoagentx/evo_modules/lesson_generator.py`
- `/Users/sss/directionai/DirectionAICloud/evoagentx/evo_modules/lesson_plan_evaluator.py`
- `/Users/sss/directionai/DirectionAICloud/evoagentx/evo_modules/faiss_toolkit.py`

## 3. 第一阶段：定义 schema

先完成：

- `lesson_schemas.py`
- `task_specs.py`
- `artifacts.py`
- `states.py`

完成标准：

- request schema 清楚
- artifact schema 清楚
- reviewer 输出 schema 清楚

## 4. 第二阶段：接兼容层

先实现：

- `legacy_request_mapper.py`
- `legacy_response_mapper.py`
- `sse_event_mapper.py`
- `lesson_router.py`

完成标准：

- 能接收旧前端 lesson 请求
- 能稳定输出 lesson SSE
- `done.lesson_plan` 能返回

## 5. 第三阶段：实现核心 workflow

实现：

- `lesson_agents.py`
- `lesson_workflow.py`
- `lesson_service.py`
- `lesson_artifact_builder.py`

完成标准：

- planner / grounder / pedagogy / activity / reviewer 流转可运行
- 产物不是纯自由文本

## 6. 第四阶段：接知识增强

实现：

- RAG tool wrapper
- document parser wrapper
- lesson validation tool

完成标准：

- `use_rag=true` 和 `use_rag=false` 都能跑
- 检索失败时能降级

## 7. 第五阶段：测试

必须补：

- contract test
- integration test
- regression test

重点测试用例：

- 最小输入
- 长约束输入
- use_rag=true
- use_rag=false
- reviewer fail 回炉
- 最终 downstream_context 可用于下游

## 8. DoD

教案链完成标准：

- 旧前端 lesson 页面无需大改
- 新后端 lesson 链不再依赖旧 workflow JSON 才能工作
- lesson 输出有稳定 artifact
- SSE contract 稳定
- 可作为试卷和 PPT 上游
- 有基础回归测试
