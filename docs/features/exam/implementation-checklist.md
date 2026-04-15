# 试卷功能实施清单

## 1. 迁移来源

优先阅读：

- `/Users/sss/directionai/DirectionAICloud/evoagentx/evo_modules/exam_generator.py`
- `/Users/sss/directionai/DirectionAICloud/evoagentx/evo_modules/question_evaluator.py`

## 2. 第一阶段：定义 exam schema

先补：

- `exam_schemas.py`
- `task_specs.py`
- `artifacts.py`

## 3. 第二阶段：兼容层

先补：

- `legacy_request_mapper.py`
- `legacy_response_mapper.py`
- `sse_event_mapper.py`
- `exam_router.py`

完成标准：

- 旧请求可进入新链
- `done` 能兼容旧前端字段

## 4. 第三阶段：blueprint + fan-out

实现：

- `ExamBlueprintPlanner`
- 各题型 generator
- assembler

## 5. 第四阶段：checker / dedup

实现：

- answer / analysis checker
- dedup reviewer
- 局部修复逻辑

## 6. 第五阶段：测试

必须补：

- contract tests
- integration tests
- regression tests

重点用例：

- 多种题型组合
- 零题型
- 难度比例
- 重复题检测
- `thinking_chunk` JSON 路由
- `done` 兼容字段

## 7. DoD

- 旧前端可继续消费试卷结果
- fan-out 正常
- checker / dedup 可运行
- 输出结构稳定
- 回归测试具备
