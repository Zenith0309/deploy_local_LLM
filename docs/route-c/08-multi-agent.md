# 8. 多 Agent 协作

## 1. 什么时候值得拆分

拆分的理由应该是权限、工具、上下文或专业职责明显不同，而不是为了让架构看起来复杂。适合拆分的角色：

```text
规划 Agent：拆分任务
检索 Agent：查找证据
执行 Agent：调用业务工具
审核 Agent：检查结果
```

## 2. 常见拓扑

- **主管-工人**：主管分配任务，工人返回结果。
- **流水线**：每个 Agent 负责一个阶段。
- **专家辩论**：多个 Agent 给出观点，审核 Agent 汇总。
- **共享黑板**：所有 Agent 读写受控任务状态。

## 3. 多 Agent 的成本

多 Agent 会增加：

- 消息和上下文成本。
- 状态一致性问题。
- 权限传播风险。
- 调试和回放难度。
- 循环和互相调用风险。

先用单 Agent 完成任务并建立评测基线，再比较多 Agent 是否真的提高质量。

## 4. 零信任协作

每次 Agent 间通信都要验证发送者、接收者、任务范围和数据权限。一个研究 Agent 发现的信息不能自动获得写入或支付权限。

### 中文参考

- [Multi-Agent 多智能体协作系统指南（腾讯云开发者社区）](https://cloud.tencent.com/developer/article/2649756)
- [从 0 到 1 搭建 AI Agent 工作流（阿里云开发者社区）](https://developer.aliyun.com/article/1709722)
- [Agent 完整教程](../knowledge/agents.md)

[返回路线 C 总览](README.md) · [下一篇：模型与应用评测](09-evaluation.md)
