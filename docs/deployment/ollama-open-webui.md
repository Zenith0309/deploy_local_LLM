# Ollama + Open WebUI：本地部署大模型完整教程

本教程从一台普通个人电脑开始，完成：

```text
硬件检查 → 安装 Ollama → 下载模型 → 终端对话
      → API 验证 → Docker 部署 Open WebUI → 性能调优 → 排障和安全
```

Ollama 负责模型下载、模型文件管理和推理服务；Open WebUI 负责提供浏览器界面。二者分开运行，便于排查问题：先确保 Ollama 和 API 正常，再安装 Web UI。

## 1. 先理解部署架构

最小架构如下：

```text
浏览器 / Python / Node.js
          │ HTTP
          ▼
Ollama API: localhost:11434
          │
          ▼
本地模型文件 + CPU/GPU 推理
```

加入 Open WebUI 后：

```text
浏览器
  │ http://localhost:3000
  ▼
Open WebUI 容器
  │ http://host.docker.internal:11434
  ▼
宿主机 Ollama
  │
  ▼
本地模型
```

容器中的 `localhost` 指向容器自身，不等于宿主机。因此 WebUI 容器连接宿主机 Ollama 时，通常不能填写 `http://localhost:11434`。

## 2. 环境检查与资源估算

### 2.1 操作系统

支持 Windows、macOS 和 Linux。建议使用 64 位系统，并保持系统、显卡驱动和 Docker Desktop 为较新的稳定版本。

### 2.2 内存、显存和磁盘

模型的磁盘大小、加载内存和运行时显存不是同一个数字：

- **磁盘**：模型下载后占用的文件空间。
- **内存/显存**：模型加载后的权重、KV Cache、运行时缓冲区。
- **上下文**：对话越长，KV Cache 通常越大。
- **并发**：同时请求越多，运行时占用通常越高。

经验起点：

| 设备资源 | 建议模型规模 | 适合场景 |
| --- | --- | --- |
| 8 GB 内存、无独显 | 1B-3B | 简单问答、摘要、轻量代码 |
| 16 GB 内存、无独显 | 3B-7B 量化 | 中文问答、文本处理 |
| 6-8 GB 显存 | 7B/8B 量化 | 交互式对话、基础代码 |
| 12-16 GB 显存 | 7B/14B 量化 | 更好的质量和上下文 |
| 24 GB 以上显存 | 14B 及以上 | 更复杂任务或更高吞吐 |

这不是硬性公式。实际能力还受模型架构、量化格式、上下文长度、GPU 是否被其他程序占用和系统驱动影响。

### 2.3 检查可用空间

模型文件可能从数 GB 到数十 GB。下载前至少确认：

```text
模型文件预计大小
+ 临时下载空间
+ 运行时缓存
+ Open WebUI 数据
+ 系统剩余空间
```

不要把系统盘填满。模型存储位置应放在空间充足、读写性能较好的磁盘上。

## 3. 安装 Ollama

从 [Ollama 官网](https://ollama.com/) 下载对应平台的安装程序。安装后重新打开终端，验证：

```bash
ollama --version
```

检查服务是否可访问：

```bash
curl http://localhost:11434/api/tags
```

Windows PowerShell：

```powershell
Invoke-RestMethod -Uri "http://localhost:11434/api/tags"
```

如果能够返回模型列表 JSON，即使列表为空，也说明 Ollama API 已经监听。

### 命令找不到怎么办

1. 关闭并重新打开终端。
2. 确认安装程序已经完成。
3. 检查 Ollama 是否加入 PATH。
4. Windows 检查 Ollama 应用是否在后台运行。
5. Linux/macOS 检查系统服务或手动启动 `ollama serve`。

不要同时启动多个 Ollama 服务实例，否则可能产生端口冲突或使用不同的模型目录。

## 4. 选择模型

不要只看模型参数量。选择模型时至少比较：

1. **任务类型**：通用对话、代码、数学、视觉、嵌入或语音。
2. **模型标签**：同一模型的不同大小和量化版本资源差异很大。
3. **上下文长度**：长上下文会增加内存/显存使用。
4. **语言能力**：中文、英文、代码和专业领域表现可能不同。
5. **许可证和使用限制**：用于商业应用前必须阅读模型许可。
6. **速度与质量**：小模型可能更快，大模型可能更准确，但不存在绝对最优。

第一次部署建议从 3B-7B 级量化模型开始，而不是直接下载最大版本。下面使用 `qwen2.5:7b` 作为示例标签；执行前请到 [Ollama 模型库](https://ollama.com/library) 确认标签仍然可用。

## 5. 下载并运行第一个模型

```bash
ollama run qwen2.5:7b
```

首次执行会下载模型。下载过程中：

- 不要关闭终端或强制结束 Ollama。
- 确认磁盘空间没有耗尽。
- 网络中断后可以重新执行相同命令。
- 不要因为终端暂时没有输出就重复创建多个下载任务。

进入对话后测试：

```text
请用中文回答：什么是本地大模型？请分别说明模型、推理服务和聊天界面的作用。
```

退出：

```text
/bye
```

管理模型：

```bash
# 列出本地模型
ollama list

# 查看模型当前是否在运行
ollama ps

# 查看模型信息
ollama show qwen2.5:7b

# 删除模型
ollama rm qwen2.5:7b
```

删除模型会释放模型文件空间，但不会自动清理其他应用的缓存。

## 6. 配置和管理模型

### 6.1 Modelfile 的用途

如果你希望固定系统提示词、采样参数或模板，可以使用 Modelfile：

```text
FROM qwen2.5:7b

SYSTEM """
你是一个严谨的本地知识库助手。
如果没有足够证据，请明确说不知道，不要编造事实。
"""

PARAMETER temperature 0.2
PARAMETER num_ctx 4096
```

保存为 `Modelfile` 后创建模型：

```bash
ollama create local-assistant -f Modelfile
ollama run local-assistant
```

注意：系统提示词不能替代权限控制和事实验证。它只是模型上下文的一部分。

### 6.2 参数的基本含义

- `temperature`：随机性。较低通常更稳定，较高可能更有创造性。
- `num_ctx`：上下文窗口大小。越大通常越占内存/显存。
- `top_p`、`top_k`：影响候选 token 的采样范围。
- `num_predict`：限制生成长度，避免一次输出过长。

调参时一次只改一个变量，并用固定测试问题比较结果。

## 7. 验证 Ollama API

### 7.1 查看模型列表

Linux/macOS：

```bash
curl http://localhost:11434/api/tags
```

Windows PowerShell：

```powershell
Invoke-RestMethod -Uri "http://localhost:11434/api/tags"
```

### 7.2 非流式生成

Linux/macOS：

```bash
curl http://localhost:11434/api/generate \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen2.5:7b",
    "prompt": "用一句话解释什么是本地大模型。",
    "stream": false,
    "options": {
      "temperature": 0.2,
      "num_ctx": 4096
    }
  }'
```

Windows PowerShell：

```powershell
$body = @{
  model = "qwen2.5:7b"
  prompt = "用一句话解释什么是本地大模型。"
  stream = $false
  options = @{
    temperature = 0.2
    num_ctx = 4096
  }
} | ConvertTo-Json -Depth 4

Invoke-RestMethod `
  -Uri "http://localhost:11434/api/generate" `
  -Method Post `
  -ContentType "application/json" `
  -Body $body
```

成功判据：

- HTTP 请求成功。
- 返回 JSON。
- 返回对象包含 `response`。
- 服务没有在生成期间崩溃或持续重启。

### 7.3 对话接口

对多轮对话使用 `/api/chat`：

```bash
curl http://localhost:11434/api/chat \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen2.5:7b",
    "stream": false,
    "messages": [
      {"role": "system", "content": "你是一个简洁的技术助手。"},
      {"role": "user", "content": "什么是 Harness？"},
      {"role": "assistant", "content": "Harness 是控制 Agent 执行的工程层。"},
      {"role": "user", "content": "它为什么需要状态管理？"}
    ]
  }'
```

不要无限增长 `messages`。生产应用需要截断历史、摘要或使用外部记忆，否则上下文会越来越长。

## 8. 用 Python 调用本地模型

安装 `requests`：

```bash
python -m pip install requests
```

示例：

```python
import requests

payload = {
    "model": "qwen2.5:7b",
    "prompt": "列出本地部署大模型时最重要的三个检查项。",
    "stream": False,
}

response = requests.post(
    "http://localhost:11434/api/generate",
    json=payload,
    timeout=120,
)
response.raise_for_status()
data = response.json()
print(data["response"])
```

应用代码必须设置超时、检查 HTTP 状态码和处理 JSON 错误。不要把模型返回内容直接当成可信命令执行。

## 9. 使用 Open WebUI

先确认 Docker Desktop 或 Docker Engine 已启动。Open WebUI 的数据保存在 Docker volume 中，因此删除容器不会自动删除聊天数据。

### Linux 和 macOS

```bash
docker run -d \
  --name open-webui \
  -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  --restart unless-stopped \
  ghcr.io/open-webui/open-webui:main
```

### Windows PowerShell

```powershell
docker run -d `
  --name open-webui `
  -p 3000:8080 `
  --add-host=host.docker.internal:host-gateway `
  -v open-webui:/app/backend/data `
  --restart unless-stopped `
  ghcr.io/open-webui/open-webui:main
```

浏览器打开 <http://localhost:3000>，首次访问时创建本地账号。进入设置后，如果没有自动发现 Ollama，将连接地址设置为：

```text
http://host.docker.internal:11434
```

Linux 上如果该地址不可用，检查 Docker 到宿主机的网络配置，确认 Ollama 的监听地址和防火墙规则。不要为了临时解决连接问题而把服务暴露到公网。

容器管理：

```bash
docker ps
docker logs -f open-webui
docker stop open-webui
docker start open-webui
docker rm -f open-webui
```

彻底删除 WebUI 数据：

```bash
docker volume rm open-webui
```

这个命令会删除聊天记录和 WebUI 配置，执行前确认不再需要这些数据。

## 10. 性能调优方法

### 10.1 先测量，不要凭感觉

固定一组测试问题，记录：

- 首 token 延迟。
- 生成速度。
- 总响应时间。
- 模型加载时间。
- CPU、内存、GPU 和显存占用。
- 相同问题下的答案质量。

每次只改变一个因素，例如模型大小、上下文、温度或并发数。

### 10.2 推理慢的排查顺序

1. 确认模型是否太大。
2. 确认后台是否有其他程序占用内存或显存。
3. 降低上下文长度。
4. 减少同时运行的模型数量。
5. 检查显卡驱动和 Ollama 是否正确使用 GPU。
6. 使用更小或更适合硬件的量化模型。

CPU 推理慢通常是资源限制，不一定是配置错误。

### 10.3 上下文和并发

上下文越长，模型可参考的信息越多，但 KV Cache 占用也会增加。并发请求会争夺计算和内存资源。个人电脑建议先使用单并发，确认稳定后再逐步增加。

## 11. 安全与隐私

### 11.1 默认只监听本机

本地 API 不应默认对局域网或公网开放。确实需要远程访问时，至少增加身份认证、TLS、网络访问控制、限流和日志审计。

### 11.2 不要把敏感内容写入日志

聊天内容、文件内容、API Key 和个人数据可能进入应用日志。记录调试信息时应脱敏，并提供日志保留期限和清理方法。

### 11.3 本地模型也会被提示词注入

来自网页、文档和用户上传文件的内容可能包含“忽略原指令”之类的文本。将外部内容视为不可信数据，不要让它直接改变工具权限。

## 12. 常见问题与诊断流程

### `ollama: command not found`

重新打开终端，确认安装和 PATH。若服务与命令行客户端版本不一致，重新安装同一版本。

### `model not found`

执行 `ollama list`，检查名称和标签是否完全一致。不要根据记忆猜模型标签。

### 下载失败或速度慢

检查网络、代理和磁盘空间。重新运行相同的 `ollama run`，不要重复创建模型。必要时先删除不完整文件并重试。

### `out of memory`

按以下顺序处理：

1. 换小模型。
2. 降低 `num_ctx`。
3. 关闭其他模型和占用显存的程序。
4. 减少并发。
5. 改用更适合当前硬件的量化版本。

### API 连接被拒绝

确认 Ollama 正在运行，执行 `/api/tags`。检查是否有其他程序占用 `11434`，以及应用是否错误地连接了容器内的 `localhost`。

### Open WebUI 无法连接 Ollama

先从宿主机验证 Ollama API，再检查容器日志：

```bash
docker logs open-webui
```

容器连接宿主机通常使用 `host.docker.internal`。如果地址不可用，检查 Docker 网络和宿主机防火墙。

### 端口 `3000` 已被占用

把左侧宿主机端口改成其他端口，例如 `-p 3001:8080`，然后访问 `http://localhost:3001`。

## 13. 清理和卸载

停止并删除 WebUI：

```bash
docker rm -f open-webui
```

确认不再需要后删除数据卷：

```bash
docker volume rm open-webui
```

删除模型：

```bash
ollama list
ollama rm <模型名称>
```

卸载 Ollama 前，先备份需要保留的模型配置、Modelfile 和应用代码。

## 14. 下一步

- [Agent 完整教程](../knowledge/agents.md)：让模型通过工具完成多步任务。
- [Harness 完整教程](../knowledge/harness.md)：为 Agent 增加权限、状态、预算、恢复和审计。
- [部署教程索引](README.md)：继续学习 GPU、vLLM、RAG 和生产部署。

[返回教程主页](../../README.md)
