# 本地部署大模型教程

从零开始在个人电脑上运行大模型，并进一步了解 Agent、Harness 和本地 AI 应用的工程实践。

## 快速开始

如果你是第一次部署，按下面的顺序阅读：

1. [本地部署总教程：Ollama + Open WebUI](docs/deployment/ollama-open-webui.md)
2. [Agent 基础与实践](docs/knowledge/agents.md)
3. [Harness 基础与实践](docs/knowledge/harness.md)

完成第一篇后，你应该能够在本机运行模型、通过浏览器对话，并使用 HTTP API 调用模型。

## 教程导航

### 部署教程

| 教程 | 适合人群 | 内容 |
| --- | --- | --- |
| [Ollama + Open WebUI](docs/deployment/ollama-open-webui.md) | 初学者 | 最短路径运行本地模型和 Web UI |
| [部署教程索引](docs/deployment/README.md) | 所有人 | 查看部署路线和后续专题规划 |

### 知识教程

| 教程 | 适合人群 | 内容 |
| --- | --- | --- |
| [Agent 基础与实践](docs/knowledge/agents.md) | 想构建 AI Agent 的开发者 | Agent 的组成、工具调用、记忆、规划和安全 |
| [Harness 基础与实践](docs/knowledge/harness.md) | 想让 Agent 稳定运行的开发者 | Harness 的职责、执行循环、状态管理、验证和恢复 |
| [知识教程索引](docs/knowledge/README.md) | 所有人 | 查看概念和工程实践专题 |

## 推荐学习路径

```text
本地模型运行
      ↓
API 调用与提示词
      ↓
Agent：让模型使用工具完成任务
      ↓
Harness：让 Agent 可控、可观测、可恢复
      ↓
RAG、工作流、多 Agent 和生产化部署
```

## 这套教程的原则

- **先跑通，再深入**：先用 Ollama 跑通最小闭环，再学习更复杂的推理服务。
- **命令可复制**：每个教程都给出命令、预期结果和常见故障。
- **区分概念**：模型负责生成，Agent 负责决策，工具负责执行，Harness 负责约束和编排。
- **本地优先**：示例优先考虑数据隐私、可离线运行和个人电脑资源限制。
- **从可验证开始**：涉及 Agent 的示例都应有明确的成功判据、超时和失败处理。

## 工具与参考资料

- [Ollama](https://ollama.com/)
- [Ollama 模型库](https://ollama.com/library)
- [Open WebUI](https://github.com/open-webui/open-webui)
- [vLLM](https://github.com/vllm-project/vllm)

## 贡献教程内容

新增教程时，请说明适用操作系统、硬件要求、模型版本、可复制命令、预期输出和排障方法。新增文档后，把链接补充到对应的索引页和本 README。
