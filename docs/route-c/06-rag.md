# 6. RAG 检索增强生成

## 1. RAG 解决什么问题

RAG 让模型在回答前检索外部资料，适合本地文档、企业知识、产品手册和会持续更新的数据。它不能自动解决复杂推理、错误文档和检索质量问题。

```text
文档 → 清洗 → 分块 → Embedding → 向量索引
问题 → Embedding → 召回 → 重排 → 上下文
                                      ↓
                                本地模型生成
```

## 2. 最小实现步骤

1. 选择 Markdown、PDF 或数据库作为数据源。
2. 清洗页眉、页脚、重复内容和敏感信息。
3. 按语义和长度分块，保留标题和来源元数据。
4. 使用 Embedding 模型生成向量。
5. 写入向量数据库或本地索引。
6. 对用户问题进行向量召回。
7. 必要时结合关键词检索和重排。
8. 把有限片段注入提示词。
9. 要求模型引用来源，不足时拒答。

## 3. 质量不能只看最终回答

分层检查：

- 召回：相关文档是否被找到。
- 排序：最相关片段是否靠前。
- 上下文：片段是否完整、无冲突。
- 生成：答案是否忠实于片段。
- 端到端：是否真正解决用户任务。

## 4. 本地实践建议

先使用一个小型 Markdown 知识库，保存 `source`、标题、更新时间和权限信息。不要把不同用户的数据混进同一个无隔离索引。

### 中文参考

- [All-in-RAG：RAG 技术全栈指南（Datawhale）](https://datawhalechina.github.io/all-in-rag/)
- [All-in-RAG GitHub 仓库](https://github.com/datawhalechina/all-in-rag)
- [使用 MongoDB Vector Search 构建 RAG（中文官方文档）](https://www.mongodb.com/zh-cn/docs/vector-search/tutorials/rag/)
- [RAG 全景与生产实践（腾讯云开发者社区）](https://cloud.tencent.com/developer/article/2654878)

[返回路线 C 总览](README.md) · [下一篇：Agent 工作流](07-agent-workflows.md)
