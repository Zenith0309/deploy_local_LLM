# 4. Agent 工具调用与任务执行

本篇是 [Agent 完整教程](../knowledge/agents.md) 的路线 C 实践入口。

## 1. 先从只读工具开始

一个安全的第一个工具可以是：

```text
search_documents(query, limit)
```

它只读取固定知识库，不修改文件、不发网络请求、不执行任意命令。

工具需要定义：

- 名称和用途。
- 参数类型、范围和必填项。
- 成功和错误返回结构。
- 是否只读。
- 资源范围和超时。

## 2. ReAct 与 Function Calling

典型循环：

```text
用户目标 → 模型选择工具 → Harness 校验
        → 工具执行 → 观察结果 → 模型继续判断
        → 最终答案
```

模型只提出工具调用，真正的执行权在应用和 Harness。工具结果应作为数据回传，不能被解释成新的系统规则。

## 3. 工具安全

避免直接暴露无限制 Shell。若必须执行命令，至少使用白名单、固定目录、低权限用户、超时、输出限制和网络隔离。

写操作还需要：

- 显式审批。
- 幂等键。
- 审计日志。
- 可回滚或补偿动作。

## 4. Agent 验收

固定测试：

1. 正常查询能选择正确工具。
2. 缺参数时能返回明确错误。
3. 工具超时时能停止。
4. 未授权工具会被拦截。
5. 没有证据时不会伪造答案。

### 中文参考

- [Agent 节点中文文档（Dify）](https://docs.dify.ai/zh/self-host/use-dify/nodes/agent)
- [从 0 到 1 搭建 AI Agent 工作流（阿里云开发者社区）](https://developer.aliyun.com/article/1709722)
- [Agent 基础与实践](../knowledge/agents.md)

[返回路线 C 总览](README.md) · [下一篇：Harness 工程化控制](05-harness-engineering.md)
