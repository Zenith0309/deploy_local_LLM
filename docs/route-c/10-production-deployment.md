# 10. 生产部署与可观测性

## 1. 从本地实验到服务

生产架构通常至少包括：

```text
客户端
  ↓
API 网关：鉴权、限流、配额
  ↓
应用/Harness：任务、工具、状态和审计
  ↓
推理服务：Ollama、vLLM 或其他引擎
  ↓
GPU/CPU 节点和模型存储
```

RAG 还需要文档处理、Embedding、索引和权限隔离组件。

## 2. 发布前检查

```text
[ ] API 身份认证和授权
[ ] 请求、输出和文件大小限制
[ ] 超时、取消、重试和幂等
[ ] 敏感数据脱敏和日志保留策略
[ ] 模型、提示词、工具和策略版本
[ ] 固定评测集通过
[ ] GPU/内存/磁盘告警
[ ] 回滚和模型切换方案
[ ] 备份和恢复演练
```

## 3. 关键监控指标

- 请求量、成功率和错误码。
- 首 token 延迟、总延迟和 P95/P99。
- 输入输出 token 数和上下文长度。
- GPU 利用率、显存、CPU、内存和温度。
- KV Cache、队列长度和并发数。
- 工具调用成功率、重试和审批等待时间。
- RAG 召回质量和引用缺失。

监控不仅要看“服务是否活着”，还要看回答质量和工具行为是否退化。

## 4. 扩展策略

个人电脑优先保持单实例。需要扩展时再考虑：

- 多模型路由。
- 请求队列和批处理。
- GPU 节点扩容。
- Kubernetes 和 Helm。
- Prometheus、Grafana 和集中式日志。
- 灰度发布和 A/B 测试。

### 中文参考

- [vLLM Production Stack 中文参考栈](https://github.com/vllm-project/production-stack/blob/main/README.zh.md)
- [vLLM Ascend 中文部署指南](https://docs.vllm.ai/projects/vllm-ascend-cn/zh-cn/latest/user_guide/deployment_guide/)
- [基于 vLLM 的大模型服务监控体系（百度开发者中心）](https://developer.baidu.com/article/detail.html?id=5596215)
- [本地部署与安全章节](../deployment/ollama-open-webui.md#11-安全与隐私)

[返回路线 C 总览](README.md) · [返回教程主页](../../README.md)
