# 2. 本地模型与推理服务

## 目标

理解“模型文件”和“推理服务”的区别，并完成一次稳定的 API 调用。详细的 Ollama + Open WebUI 操作见[完整部署教程](../deployment/ollama-open-webui.md)，本篇强调架构和服务化边界。

## 1. 三层架构

```text
应用：浏览器、Python、Node.js、Agent
             ↓ HTTP
推理服务：Ollama、llama.cpp server、vLLM
             ↓
模型权重：GGUF、GPTQ、AWQ、Safetensors 等
```

应用不应该直接读取模型权重，也不应该把 GPU 管理逻辑散落在业务代码中。

## 2. 使用 Ollama 建立基线

```bash
ollama --version
ollama run qwen2.5:7b
ollama list
ollama ps
```

用 API 验证：

```bash
curl http://localhost:11434/api/generate \
  -H "Content-Type: application/json" \
  -d '{"model":"qwen2.5:7b","prompt":"用一句话介绍本地推理服务。","stream":false}'
```

成功判据：

- API 返回 HTTP 成功。
- JSON 中有 `response`。
- 模型没有因为内存或显存不足退出。
- 第二次请求的加载时间明显低于第一次，或符合预期。

## 3. API 应用的基本防护

客户端应设置：

- 连接超时和读取超时。
- HTTP 状态码检查。
- JSON Schema 或字段存在检查。
- 最大输入长度和最大输出长度。
- 取消请求和重试策略。

不要把模型返回内容直接拼接成 Shell 命令、SQL 或文件路径。

## 4. 什么时候换 vLLM

Ollama 适合个人电脑、开发验证和低并发。以下情况可以评估 vLLM：

- Linux + NVIDIA GPU 服务端。
- 需要 OpenAI 兼容 API。
- 需要连续批处理和更高吞吐。
- 需要多实例、路由、监控和 Kubernetes 部署。

不要为了“看起来生产级”而在只有一台个人电脑时过早引入复杂集群。

### 中文参考

- [本地部署总教程：Ollama + Open WebUI](../deployment/ollama-open-webui.md)
- [vLLM Production Stack 中文 README](https://github.com/vllm-project/production-stack/blob/main/README.zh.md)
- [vLLM Ascend 中文部署指南](https://docs.vllm.ai/projects/vllm-ascend-cn/zh-cn/latest/)

[返回路线 C 总览](README.md) · [上一篇](01-hardware-and-model-selection.md) · [下一篇](03-prompt-context-and-structured-output.md)
