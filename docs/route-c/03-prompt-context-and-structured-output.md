# 3. 提示词、上下文与结构化输出

## 目标

本篇解决三个问题：

1. 如何把模糊需求改写成可执行任务。
2. 如何控制上下文，避免信息过多或缺失。
3. 如何让输出满足 JSON、表格或业务 Schema。

## 1. 提示词的基本结构

推荐按以下顺序组织：

```text
角色：你是谁
目标：要完成什么
输入：处理哪些数据
约束：不能做什么、边界是什么
步骤：需要遵循哪些检查
输出：字段、类型、格式和完成条件
```

示例：

```text
你是本地部署排障助手。
根据用户提供的日志，判断最可能的原因。
只能使用日志中出现的证据，不要猜测不存在的配置。
输出 JSON，字段为 cause、evidence、next_steps、confidence。
如果证据不足，confidence 必须低于 0.5。
```

## 2. 上下文工程

上下文可以包括系统指令、用户目标、历史消息、工具定义、检索片段和工具结果。控制原则：

- 只放当前任务需要的信息。
- 对旧对话做摘要。
- 为外部文档标注来源和不可信边界。
- 不要把所有文档全部塞入窗口。
- 工具结果限制大小和字段。

上下文越长不一定越好，过长会增加延迟、资源占用和模型混淆。

## 3. 结构化输出

先定义 Schema，再写提示词：

```json
{
  "type": "object",
  "properties": {
    "answer": {"type": "string"},
    "sources": {"type": "array", "items": {"type": "string"}},
    "confidence": {"type": "number", "minimum": 0, "maximum": 1}
  },
  "required": ["answer", "sources", "confidence"],
  "additionalProperties": false
}
```

解析流程：

```text
模型输出 → JSON 解析 → Schema 校验 → 业务校验 → 进入下一步
```

解析失败时可以有限次数地要求模型修正，但不能无限重试，也不能把非法输出当成成功。

## 4. 参数如何调

- `temperature`：事实抽取和分类通常使用较低值；创意任务可以提高。
- `num_ctx`：只设到业务需要的大小。
- `num_predict`：限制最大输出，防止异常长回复。

一次只修改一个参数，并用固定测试集比较。

### 中文参考

- [大模型提示词工程完全指南（百度开发者中心）](https://developer.baidu.com/article/detail.html?id=5541058)
- [提示工程与上下文工程解析（百度开发者中心）](https://developer.baidu.com/article/detail.html?id=7428604)
- [大模型应用开发实战：从 Prompt 到企业级 RAG（腾讯云开发者社区）](https://cloud.tencent.com/developer/article/2588835)

[返回路线 C 总览](README.md) · [上一篇](02-local-model-and-inference.md) · [下一篇](04-agent-tool-use.md)
