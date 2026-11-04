# Docker 部署本地大模型：Ollama + Open WebUI

本教程使用 Docker Compose 把 Ollama 和 Open WebUI 部署成一套可重复启动、可持久化、易于清理的本地服务。

完成后，系统结构如下：

```text
浏览器
   │ http://localhost:3000
   ▼
Open WebUI 容器
   │ http://ollama:11434
   ▼
Ollama 容器
   │
   ├── CPU 或 GPU 推理
   └── ollama-data volume：模型文件
```

与“宿主机安装 Ollama、容器运行 WebUI”的方式相比，本教程把两个服务都放进 Docker 网络中。服务之间通过 Compose 服务名 `ollama` 通信，模型文件和 WebUI 数据通过 named volume 持久化。

> **适用范围**：Windows Docker Desktop、Linux Docker Engine、macOS Docker Desktop。NVIDIA GPU 需要额外完成宿主机驱动和 NVIDIA Container Toolkit 配置。

## 1. 为什么使用 Docker

Docker 适合以下场景：

- 希望隔离运行环境，不在宿主机安装多个依赖。
- 希望通过一个 Compose 文件启动或停止整套服务。
- 希望把模型文件、WebUI 数据和配置明确分开。
- 希望以后迁移到另一台机器或服务器。
- 希望快速销毁实验环境并重新创建。

Docker 不会自动解决硬件问题。容器仍然使用宿主机的 CPU、内存、磁盘和 GPU，模型下载速度也仍然取决于网络。

## 2. 前置要求

### 2.1 安装 Docker

Windows 和 macOS 安装 [Docker Desktop](https://www.docker.com/products/docker-desktop/)，Linux 安装 Docker Engine 和 Docker Compose 插件。

检查版本：

```bash
docker --version
docker compose version
```

检查 Docker 是否可以启动容器：

```bash
docker run --rm hello-world
```

如果 `hello-world` 能够输出欢迎信息，说明 Docker CLI、Docker Engine 和镜像拉取基本正常。

### 2.2 资源建议

除了模型本身，还要为 Docker、Open WebUI 和操作系统保留资源：

| 资源 | 建议 |
| --- | --- |
| 内存 | 16 GB 更适合运行 7B/8B 量化模型 |
| 磁盘 | 至少预留 30 GB，多个模型需要更多空间 |
| CPU | 4 核以上更适合本地交互 |
| GPU | 非必需；NVIDIA GPU 需要驱动和容器运行时支持 |
| 网络 | 首次拉取镜像和模型需要稳定网络 |

Docker Desktop 用户还需要在 Docker Desktop 的资源设置中确认虚拟机获得了足够的内存和磁盘空间。

## 3. 创建项目目录

Linux/macOS：

```bash
mkdir local-llm-docker
cd local-llm-docker
```

Windows PowerShell：

```powershell
New-Item -ItemType Directory -Force local-llm-docker | Out-Null
Set-Location local-llm-docker
```

后续的 `docker compose` 命令都在这个目录执行。

## 4. 创建 CPU 版 Compose 配置

创建 `compose.yaml`：

```yaml
services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    restart: unless-stopped
    volumes:
      - ollama-data:/root/.ollama
    networks:
      - llm-net

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: unless-stopped
    depends_on:
      - ollama
    ports:
      - "3000:8080"
    environment:
      OLLAMA_BASE_URL: http://ollama:11434
    volumes:
      - open-webui-data:/app/backend/data
    networks:
      - llm-net

volumes:
  ollama-data:
  open-webui-data:

networks:
  llm-net:
    driver: bridge
```

### 配置说明

- `ollama-data` 保存模型文件。删除容器不会删除这个 volume。
- `open-webui-data` 保存账号、聊天记录和 WebUI 配置。
- `OLLAMA_BASE_URL` 使用 Compose 服务名连接 Ollama，不要写容器内的 `localhost`。
- `3000:8080` 表示宿主机端口 `3000` 映射到 WebUI 容器端口 `8080`。
- `restart: unless-stopped` 会在 Docker 重启后自动恢复容器。
- `depends_on` 控制启动顺序，但不代表 Ollama API 已经完成健康检查；第一次启动时 WebUI 可能需要等待几秒后再连接。

## 5. 启动服务

```bash
docker compose up -d
```

查看容器：

```bash
docker compose ps
```

查看日志：

```bash
docker compose logs -f ollama
docker compose logs -f open-webui
```

浏览器访问：

```text
http://localhost:3000
```

首次访问时创建本地账号。不要把这个账号密码写入 Compose 文件或提交到 Git。

## 6. 下载并运行模型

Ollama 容器启动后，使用容器内的 Ollama CLI：

```bash
docker compose exec ollama ollama pull qwen2.5:7b
```

查看模型：

```bash
docker compose exec ollama ollama list
```

直接进行一次终端对话：

```bash
docker compose exec ollama ollama run qwen2.5:7b
```

退出对话：

```text
/bye
```

检查模型是否正在运行：

```bash
docker compose exec ollama ollama ps
```

### 从宿主机验证 API

CPU 版 Compose 默认没有把 Ollama API 暴露给宿主机，只让 Open WebUI 通过内部网络访问。如果希望从宿主机的 Python、curl 或其他应用调用 Ollama，在 `ollama` 服务中添加端口：

```yaml
    ports:
      - "11434:11434"
```

修改后重新创建：

```bash
docker compose up -d
```

Linux/macOS 测试：

```bash
curl http://localhost:11434/api/generate \
  -H "Content-Type: application/json" \
  -d '{
    "model": "qwen2.5:7b",
    "prompt": "用一句话解释 Docker 容器。",
    "stream": false
  }'
```

Windows PowerShell 测试：

```powershell
$body = @{
  model = "qwen2.5:7b"
  prompt = "用一句话解释 Docker 容器。"
  stream = $false
} | ConvertTo-Json

Invoke-RestMethod `
  -Uri "http://localhost:11434/api/generate" `
  -Method Post `
  -ContentType "application/json" `
  -Body $body
```

如果只需要 Open WebUI，不要暴露 `11434`，这样可以减少宿主机上的访问面。

## 7. NVIDIA GPU 部署

### 7.1 宿主机要求

GPU 不会因为写入 `compose.yaml` 就自动可用。宿主机需要：

1. 安装与 GPU 匹配的 NVIDIA 驱动。
2. Linux 安装 NVIDIA Container Toolkit。
3. Docker 能够把 GPU 设备传递给容器。
4. 驱动、CUDA 和 Ollama 镜像组合相互兼容。

先在宿主机确认：

```bash
nvidia-smi
```

再验证 Docker GPU 运行时。具体 CUDA 镜像标签要以 NVIDIA 官方文档和本机驱动支持版本为准；不要盲目复制过期的 CUDA 标签。

### 7.2 GPU Compose 配置

在 `ollama` 服务中增加 GPU 资源声明：

```yaml
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    restart: unless-stopped
    volumes:
      - ollama-data:/root/.ollama
    networks:
      - llm-net
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
```

然后重建服务：

```bash
docker compose up -d --force-recreate ollama
docker compose exec ollama ollama pull qwen2.5:7b
docker compose exec ollama ollama run qwen2.5:7b
```

检查推理期间的 GPU 使用：

```bash
nvidia-smi
```

如果 GPU 利用率始终为零，不要先修改模型参数。先检查驱动、NVIDIA Container Toolkit、Docker GPU 测试和容器日志。

### 7.3 GPU 注意事项

- 模型太大时可能部分落到 CPU，速度会明显下降。
- 上下文长度和并发会增加显存占用。
- Docker Desktop 的 GPU 支持依赖宿主机系统和 WSL2 配置。
- AMD、Intel 和 Apple Silicon 的 GPU 配置不同，不能直接使用 NVIDIA 配置。

## 8. 容器生命周期管理

```bash
# 启动
docker compose up -d

# 停止但保留容器和数据
docker compose stop

# 启动已停止的服务
docker compose start

# 停止并删除容器、网络，但保留 named volumes
docker compose down

# 查看服务状态
docker compose ps

# 查看资源占用
docker stats
```

### 删除数据前确认

下面的命令会同时删除模型和 WebUI 数据：

```bash
docker compose down -v
```

如果只想删除容器，不删除模型：

```bash
docker compose down
```

如果需要迁移，先备份 volume 或使用 Docker 官方支持的备份方式，不要直接复制正在写入的模型目录。

## 9. 版本和升级策略

开发环境可以使用 `latest` 或 `main`，但长期运行环境不建议依赖会随时变化的标签。升级前：

1. 记录当前镜像摘要或版本。
2. 备份 WebUI 数据和重要配置。
3. 保存模型列表和 Compose 文件。
4. 先在测试目录拉取新镜像。
5. 运行固定 API 和聊天测试。
6. 确认没有数据迁移问题后再切换。

拉取新镜像：

```bash
docker compose pull
docker compose up -d
```

查看镜像：

```bash
docker image ls
```

不要在没有备份的情况下同时升级 Docker、GPU 驱动、Ollama 和模型。

## 10. 网络和安全

### 10.1 只暴露必要端口

推荐只暴露：

```yaml
ports:
  - "127.0.0.1:3000:8080"
```

这样 Open WebUI 只监听本机。若写成 `3000:8080`，Docker 可能将端口绑定到所有网络接口，局域网中的其他设备可能访问到它。

如果还需要宿主机 API：

```yaml
ports:
  - "127.0.0.1:11434:11434"
```

除非已经配置认证、TLS、反向代理和防火墙，否则不要绑定到 `0.0.0.0` 或直接映射公网。

### 10.2 不要使用特权容器

不要为了“解决权限问题”随意增加：

```yaml
privileged: true
```

也不要把宿主机根目录、Docker socket 或包含密钥的目录挂载进容器。每个 volume 只挂载完成任务所需的数据。

### 10.3 模型和数据隐私

- 模型输入可能包含聊天内容、文件内容和个人信息。
- Docker volume 仍然存储在本机磁盘上，需要纳入备份和访问控制。
- 清理容器不等于清理 volume。
- 日志可能包含请求内容，不要无期限保存。

## 11. 常见问题

### `docker compose` 不存在

安装 Docker Desktop，或在 Linux 安装 Docker Compose plugin。确认使用的是 `docker compose`，而不是旧版独立命令 `docker-compose`。

### 容器启动后马上退出

查看日志：

```bash
docker compose logs ollama
docker compose logs open-webui
```

重点检查镜像拉取失败、挂载权限、端口占用、磁盘空间和配置 YAML 缩进。

### WebUI 显示无法连接 Ollama

确认 Compose 中使用：

```text
http://ollama:11434
```

不要在 Open WebUI 容器中使用 `http://localhost:11434`。然后检查：

```bash
docker compose ps
docker compose logs open-webui
docker compose logs ollama
```

### `ollama pull` 失败

检查网络、镜像运行状态、磁盘空间和模型标签：

```bash
docker compose exec ollama ollama list
docker system df
```

模型下载和 Docker 镜像下载是两件事，分别排查。

### 端口 `3000` 或 `11434` 被占用

修改宿主机端口左侧数字，例如：

```yaml
ports:
  - "3001:8080"
```

容器内部端口仍然是 `8080`，浏览器改为访问 `http://localhost:3001`。

### 模型运行很慢

依次检查：

1. 是否使用了过大的模型。
2. Docker Desktop 是否分配了足够内存。
3. 模型是否正在使用 GPU。
4. 上下文和并发是否过高。
5. 是否有其他容器或程序占用资源。

### GPU 不可用

按顺序检查 `nvidia-smi`、NVIDIA Container Toolkit、Docker GPU 测试、Compose GPU 声明和 Ollama 日志。不要仅凭 `docker compose ps` 判断 GPU 已经生效。

### 修改 Compose 后没有生效

检查最终配置：

```bash
docker compose config
```

需要重建容器时执行：

```bash
docker compose up -d --force-recreate
```

## 12. 部署验收清单

```text
[ ] docker run hello-world 成功
[ ] docker compose up -d 成功
[ ] 两个容器状态为 running
[ ] WebUI 可以打开
[ ] Ollama 能 pull 模型
[ ] ollama list 能看到模型
[ ] 浏览器可以完成一次对话
[ ] API 返回 JSON（如果暴露了 11434）
[ ] 重启 Docker 后服务能够恢复
[ ] 删除容器后数据仍然存在
[ ] 已确认端口没有暴露到公网
[ ] 已知道如何备份和删除 volume
```

## 13. 中文参考

- [Docker 官方中文文档](https://docs.docker.com/zh-cn/)
- [Docker Compose 官方文档](https://docs.docker.com/compose/)
- [Ollama Docker 官方仓库](https://hub.docker.com/r/ollama/ollama)
- [Open WebUI 官方仓库](https://github.com/open-webui/open-webui)
- [Ollama + Open WebUI 非 Docker 部署教程](ollama-open-webui.md)

[返回部署教程索引](README.md) · [返回教程主页](../../README.md)
