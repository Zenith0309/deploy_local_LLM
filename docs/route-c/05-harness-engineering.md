# 5. Harness 工程化控制

本篇是 [Harness 完整教程](../knowledge/harness.md) 的路线 C 实践入口。

## 1. Harness 要控制什么

最小 Harness 需要负责：

```text
任务状态 + 上下文 + 工具权限 + 最大步骤
+ 超时 + 预算 + 日志 + 输出验证
```

模型可以提出动作，但不能跳过这些控制。

## 2. 最小状态机

```text
pending → running → waiting_approval → running
                 ├→ succeeded
                 ├→ failed
                 ├→ cancelled
                 └→ timed_out
```

终态不能被重复执行。重试应创建新的尝试记录，或使用明确的幂等键。

## 3. 必须设置的限制

- 最大步骤数。
- 单次模型和工具超时。
- 总任务截止时间。
- 最大重试次数。
- 最大输入、输出和工具结果大小。
- 模型调用、CPU、内存或费用预算。

## 4. 错误分类

参数错误和权限拒绝通常不应重试；临时网络错误可以有限指数退避；有副作用的工具必须在重试前确认上次是否已经成功。

## 5. 验收标准

断开模型服务、让工具超时、取消任务、重启进程，然后检查：

- 任务是否进入正确状态。
- 资源是否释放。
- 是否保存检查点。
- 是否产生重复副作用。
- 日志是否能定位失败步骤。

### 中文参考

- [Harness 完整教程](../knowledge/harness.md)
- [Agent 节点中文文档中的迭代限制与最佳实践（Dify）](https://docs.dify.ai/zh/self-host/use-dify/nodes/agent)
- [LangChain 文档中的 Agent 与 Harness 概念](https://docs.langchain.com/oss/python/langchain/overview)

[返回路线 C 总览](README.md) · [下一篇：RAG](06-rag.md)
