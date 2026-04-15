# 试卷功能需求文档

## 1. 文档定位

这份文档定义试卷链应该做什么。

如果以后出现以下变化，必须先改本文件：

- 用户可见题型行为变化
- 输出结果语义变化
- 难度分布要求变化
- 验收标准变化

## 2. 功能目标

试卷链的目标是：根据课程、知识点、题型数量、难度比例、附加要求，生成一套结构化试卷，并携带答案、解析、知识点归属和难度标注。

核心目标包括：

1. 可按题型配置生成
2. 可按难度配置生成
3. 输出结构稳定
4. 便于评估、去重、回归测试

## 3. 用户可见输入

根据旧系统现状，至少应支持这些业务字段语义：

- `subject`
- `knowledge_bases`
- `constraint`
- `language`
- `single_choice_num`
- `multiple_choice_num`
- `true_false_num`
- `fill_blank_num`
- `short_answer_num`
- `programming_num`
- `easy_percentage`
- `medium_percentage`
- `hard_percentage`
- `use_rag`
- `model_type`
- `enable_thinking`

这些字段来自：

- `/Users/sss/directionai/DirectionAICloud/evoagentx/evo_modules/exam_generator.py`

## 4. 用户可见输出

用户至少应能看到：

- 题目列表
- 每题答案
- 每题解析
- 每题知识点
- 每题难度
- 流式生成过程

## 5. 当前前端兼容要求

旧前端试卷 store：

- `/Users/sss/directionai/DirectionAICloud/nginx/src/stores/modules/exam-generation.ts`

当前已兼容的完成态字段包括：

- `question`
- `questions`
- `output.question`

因此新后端完成态必须至少能映射出上述之一，最好统一对外提供 `questions`，再在兼容层补齐旧字段。

## 6. 当前前端已依赖的 SSE 语义

当前前端已依赖：

- `thinking_start`
- `thinking_chunk`
- `thinking_end`
- `progress`
- `done`
- `error`

另外，exam store 对 `thinking_chunk` 做了特殊兼容：

- 如果 chunk 是 JSON，且包含 `step` 和 `text`
- 会按 `step` 把文本路由到对应思考步骤

因此 exam SSE 兼容层要么继续支持这个行为，要么在兼容层做同等效果的输出。

## 7. 题型要求

至少支持：

- 单选题
- 多选题
- 判断题
- 填空题
- 简答题
- 编程题

并且：

- 数量必须尽量符合配置
- 如果某类题型数量为 0，可不生成
- 题型之间不应混乱

## 8. 难度要求

至少支持：

- easy
- medium
- hard

要求：

- 整体分布尽量符合配置
- 每道题最好能带难度标记
- checker 应能发现难度严重失衡

## 9. 知识点要求

试卷不应只是一堆题目，还应尽量明确：

- 每题考查哪个知识点
- 是否覆盖了核心知识点
- 是否出现明显重复或覆盖不足

## 10. 失败与降级要求

### 10.1 局部失败可见

如果某题型失败：

- 不应静默吞掉
- 应尽量保留其他题型结果
- 应允许局部修复

### 10.2 RAG 失败可降级

如果检索失败：

- 应允许降级继续生成
- 不应让整张试卷直接不可用

## 11. 非功能性要求

### 11.1 结构稳定

试卷链必须输出结构化结果，不能只返回一大段自由文本。

### 11.2 可评估

输出必须便于：

- 题型检查
- 数量检查
- 难度检查
- 去重检查
- 回归测试

### 11.3 可扩展

未来可以继续增加题型或 checker，但当前第一版先把基础题型稳定下来。

## 12. 验收标准

1. 旧前端可继续消费试卷结果
2. `done` 阶段至少能解析出题目数组
3. 题型数量基本符合配置
4. 结构化输出稳定
5. 有 checker / dedup 能力
6. 有 contract test、integration test、regression test
