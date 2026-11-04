# 部署教程索引

本目录关注“如何把模型运行起来”，从单机体验逐步扩展到 API 服务和生产部署。

## 已有教程

- [Ollama + Open WebUI：本地运行第一个模型](ollama-open-webui.md)
- [路线 C：从本地模型到生产级 AI 应用](../route-c/README.md)

## 推荐顺序

1. 先完成 Ollama 基础部署。
2. 再使用 Open WebUI 验证浏览器交互。
3. 通过 API 将模型接入自己的脚本或应用。
4. 根据吞吐量、显存和并发需求选择更专业的推理服务。

## 规划中的专题

- [ ] 模型选择、量化格式与资源估算
- [ ] Python 和 Node.js API 客户端
- [ ] NVIDIA GPU、CUDA 与推理性能调优
- [ ] Linux + vLLM 高吞吐部署
- [ ] RAG 应用的本地部署
- [ ] 认证、网络隔离、日志和数据隐私

路线 C 中的部署相关专题：

- [硬件评估与模型选择](../route-c/01-hardware-and-model-selection.md)
- [本地模型与推理服务](../route-c/02-local-model-and-inference.md)
- [生产部署与可观测性](../route-c/10-production-deployment.md)

[返回教程主页](../../README.md)
