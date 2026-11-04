# Ollama + Open WebUI：本地运行第一个模型

这条路线适合第一次接触本地大模型的用户。Ollama 负责下载和运行模型，Open WebUI 提供浏览器界面。

## 1. 准备环境

- 64 位 Windows、macOS 或 Linux
- 8 GB 内存可从小模型开始；16 GB 或更多内存更适合 7B/8B 级模型
- 至少 20 GB 可用磁盘空间
- 稳定网络，用于首次下载模型

显卡不是必须的。没有独立显卡时可以使用 CPU，但推理速度通常较慢。

| 设备资源 | 建议起点 |
| --- | --- |
| 8 GB 内存、无独显 | 1B-3B 级量化模型 |
| 16 GB 内存或 6-8 GB 显存 | 7B/8B 级量化模型 |
| 32 GB 内存或 12 GB 以上显存 | 更长上下文或更大的 7B/14B 模型 |

## 2. 安装并验证 Ollama

从 [Ollama 官网](https://ollama.com/) 安装对应系统版本，然后打开终端：

```bash
ollama --version
```

如果命令不存在，请重新打开终端，或检查 Ollama 是否加入系统 PATH。

## 3. 下载并运行模型

先到 [Ollama 模型库](https://ollama.com/library) 确认可用标签。下面以中文通用模型为例：

```bash
ollama run qwen2.5:7b
```

首次运行会自动下载模型。进入交互界面后输入：

```text
用三句话介绍你自己，并说明你运行在哪里。
```

退出交互界面：

```text
/bye
```

查看和删除本地模型：

```bash
ollama list
ollama rm <模型名称>
```

## 4. 验证本地 API

Ollama 默认在 `http://localhost:11434` 提供 API。保持 Ollama 运行，在另一个终端执行：

### Linux/macOS

```bash
curl http://localhost:11434/api/generate \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen2.5:7b",
    "prompt": "用一句话解释什么是本地大模型。",
    "stream": false
  }'
```

### Windows PowerShell

```powershell
$body = @{
  model = "qwen2.5:7b"
  prompt = "用一句话解释什么是本地大模型。"
  stream = $false
} | ConvertTo-Json

Invoke-RestMethod `
  -Uri "http://localhost:11434/api/generate" `
  -Method Post `
  -ContentType "application/json" `
  -Body $body
```

如果返回 JSON 且包含 `response` 字段，说明本地推理服务工作正常。

## 5. 使用 Open WebUI

先确认 Docker Desktop 或 Docker Engine 已启动。

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

浏览器打开 <http://localhost:3000>，首次访问时创建本地账号。如果没有自动发现 Ollama，在设置中使用：

```text
http://host.docker.internal:11434
```

常用容器命令：

```bash
docker stop open-webui
docker start open-webui
docker rm -f open-webui
```

## 6. 常见问题

### `model not found`

用 `ollama list` 查看本地模型名称，或到模型库复制准确的标签。

### `out of memory`

换用更小模型、降低上下文长度、关闭其他占用显存/内存的程序，并减少并发请求。

### Open WebUI 无法连接 Ollama

容器内的 `localhost` 指向容器自身，不是宿主机。优先使用 `http://host.docker.internal:11434`，并确认 Ollama 在宿主机正常运行。

### 安全提醒

除非已经配置认证、反向代理和访问控制，否则不要把 `11434` 或 Open WebUI 端口暴露到公网。

[返回部署教程索引](README.md) · [返回教程主页](../../README.md)
