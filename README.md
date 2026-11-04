# 本地部署大模型：从模型运行到 Agent Harness

这是一套面向开发者和技术爱好者的本地大模型教程。你将从一台普通个人电脑开始，完成模型下载、推理服务、Web UI、API 调用、Agent 工具调用和 Harness 工程化控制，最后具备继续学习 RAG、工作流、多 Agent 和生产化部署的基础。

> **教程定位**：优先使用可在个人电脑运行的方案，强调可复制、可验证、可恢复和数据留在本地。模型名称、镜像标签和软件命令会随版本变化，执行前请以官方文档为准。

## 你将学会什么

完成教程后，你应该能够：

- 判断自己的 CPU、内存、显存和磁盘适合什么规模的模型。
- 使用 Ollama 下载、运行、管理和调用本地模型。
- 使用 Open WebUI 搭建浏览器聊天界面。
- 使用 HTTP、Python 和 Node.js 调用本地推理服务。
- 理解模型、提示词、工具、Agent、Harness 和 RAG 的边界。
- 设计带工具调用、记忆、验证、超时和权限控制的 Agent。
- 构建一个能够暂停、恢复、重试、审计和安全失败的 Harness。
- 诊断模型下载、端口、容器连接、显存不足和推理速度问题。

## 快速开始

### 路线 A：第一次运行本地模型

1. [完整部署教程：Ollama + Open WebUI](docs/deployment/ollama-open-webui.md)
2. [部署教程索引和进阶路线](docs/deployment/README.md)

### 路线 B：想构建 Agent 应用

1. [Agent 完整教程](docs/knowledge/agents.md)
2. [Harness 完整教程](docs/knowledge/harness.md)
3. 返回部署教程，为 Agent 提供本地模型 API。

### 路线 C：想快速理解全貌

```text
硬件评估
   ↓
本地模型与推理服务
   ↓
提示词、上下文和结构化输出
   ↓
Agent：模型决定下一步并调用工具
   ↓
Harness：系统控制权限、状态、预算和恢复
   ↓
RAG、工作流、多 Agent、评测和生产部署
```

路线 C 已拆分为独立专题，建议按顺序阅读：

1. [路线 C 专题总览](docs/route-c/README.md)
2. [硬件评估、模型大小与量化选择](docs/route-c/01-hardware-and-model-selection.md)
3. [本地模型与推理服务](docs/route-c/02-local-model-and-inference.md)
4. [提示词、上下文与结构化输出](docs/route-c/03-prompt-context-and-structured-output.md)
5. [Agent 工具调用与任务执行](docs/route-c/04-agent-tool-use.md)
6. [Harness 工程化控制](docs/route-c/05-harness-engineering.md)
7. [RAG 检索增强生成](docs/route-c/06-rag.md)
8. [Agent 工作流设计](docs/route-c/07-agent-workflows.md)
9. [多 Agent 协作](docs/route-c/08-multi-agent.md)
10. [模型与应用评测](docs/route-c/09-evaluation.md)
11. [生产部署与可观测性](docs/route-c/10-production-deployment.md)

## 文档导航

### 部署教程

| 文档                                                               | 适合读者      | 主要内容                                     |
| ---------------------------------------------------------------- | --------- | ---------------------------------------- |
| [Ollama + Open WebUI 完整教程](docs/deployment/ollama-open-webui.md) | 初学者、个人开发者 | 环境检查、模型选择、Ollama、API、Docker、Web UI、调优和排障 |
| [部署教程索引](docs/deployment/README.md)                              | 所有人       | vLLM、GPU、RAG、服务化和安全专题规划                  |

### Agent 与 Harness 知识

| 文档                                        | 适合读者              | 主要内容                        |
| ----------------------------------------- | ----------------- | --------------------------- |
| [Agent 完整教程](docs/knowledge/agents.md)    | 想让模型使用工具完成任务的开发者  | 架构、工具协议、上下文、记忆、规划、循环、安全和评测  |
| [Harness 完整教程](docs/knowledge/harness.md) | 想让 Agent 稳定运行的开发者 | 生命周期、状态机、权限、预算、重试、检查点、观测和恢复 |
| [知识教程索引](docs/knowledge/README.md)        | 所有人               | 术语关系和后续知识专题                 |

## 先建立几个关键概念

### 模型不是 Agent

模型接收上下文并生成文本或结构化动作。它不会天然知道当前任务是否完成，也不会自动拥有文件、网络或系统权限。

### Agent 不是无限制自动化

Agent 是围绕目标进行多步决策的系统。真正执行动作的是工具，真正限制风险的是工具层和运行时，而不是一句系统提示词。

### Harness 是控制层

Harness 包含任务状态、上下文组装、工具路由、权限检查、预算、超时、重试、审计、验证和恢复。它把“模型可能怎么做”转化为“系统允许怎么做”。

```text
用户目标
   ↓
Harness 创建任务、设置预算和权限
   ↓
Agent/模型选择下一步
   ↓
Harness 校验工具调用
   ↓
工具执行并返回观察结果
   ↓
Harness 记录、验证、重试或结束
```

## 学习建议

- **先跑通最小闭环**：不要一开始就部署大模型、向量数据库、多 Agent 和复杂编排。
- **记录环境信息**：保存操作系统、GPU 驱动、模型标签、量化方式、上下文长度和命令。
- **所有动作都要有成功判据**：例如 API 返回 JSON、工具输出通过 Schema、任务状态变为 `succeeded`。
- **优先只读工具**：Agent 第一个工具最好是读取固定目录、查询数据或计算，而不是执行任意命令。
- **把安全放到代码里**：路径限制、网络限制、审批和预算不能只写在提示词里。
- **用固定任务回归**：升级模型或提示词后，重新运行相同任务，比较成功率和工具调用行为。

## 参考资料

- [Ollama](https://ollama.com/)
- [Ollama 模型库](https://ollama.com/library)
- [Open WebUI](https://github.com/open-webui/open-webui)
- [vLLM](https://github.com/vllm-project/vllm)
- [Hugging Face](https://huggingface.co/models)

## 贡献教程内容

新增文档时请至少说明：

1. 适用操作系统、硬件和软件版本。
2. 模型名称、标签、量化方式和预计资源占用。
3. 可直接复制的命令，以及 Windows PowerShell 和 Unix shell 的差异。
4. 预期输出、成功判据和清理方法。
5. 常见错误、诊断顺序和安全影响。
