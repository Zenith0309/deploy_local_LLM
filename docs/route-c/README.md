# 路线 C：从本地模型到生产级 AI 应用

路线 C 适合希望建立完整知识体系，而不是只运行一次聊天模型的读者。每一篇都包含核心概念、实践步骤、验收标准、常见误区和中文外部参考。

## 学习顺序

| 阶段  | 教程                                                          | 完成标准                  |
| --- | ----------------------------------------------------------- | --------------------- |
| 1   | [硬件评估、模型大小与量化选择](01-hardware-and-model-selection.md)        | 能根据资源选择模型，而不是盲目下载     |
| 2   | [本地模型与推理服务](02-local-model-and-inference.md)                | 能运行模型并用 API 调用        |
| 2a  | [Docker 部署 Ollama + Open WebUI](../deployment/docker-ollama-open-webui.md) | 能用 Compose 重复部署并持久化数据 |
| 3   | [提示词、上下文与结构化输出](03-prompt-context-and-structured-output.md) | 能设计稳定、可校验的输出契约        |
| 4   | [Agent 工具调用与任务执行](04-agent-tool-use.md)                     | 能让模型调用一个安全的只读工具       |
| 5   | [Harness 工程化控制](05-harness-engineering.md)                  | 能限制步骤、权限、超时并保存状态      |
| 6   | [RAG 检索增强生成](06-rag.md)                                     | 能从本地文档检索证据并生成带引用回答    |
| 7   | [Agent 工作流设计](07-agent-workflows.md)                        | 能把固定步骤和动态决策组合起来       |
| 8   | [多 Agent 协作](08-multi-agent.md)                             | 能判断何时拆分角色，何时保持单 Agent |
| 9   | [模型与应用评测](09-evaluation.md)                                 | 能用固定数据集比较版本，而不是凭感觉升级  |
| 10  | [生产部署与可观测性](10-production-deployment.md)                    | 能设计 API、监控、限流、发布和回滚   |

## 推荐原则

- 先完成阶段 1-3，再开始编写 Agent。
- 先使用单机、单模型、单 Agent 和只读工具。
- 每一步都保留可重复的输入、配置、输出和错误记录。
- 外部教程用于补充视角，具体命令以对应项目的官方版本文档为准。

[返回教程主页](../../README.md)
