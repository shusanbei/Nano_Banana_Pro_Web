# 🐳 Docker 部署指南

## 快速开始

### 1. 准备环境变量

```bash
# 复制环境变量模板
cp .env.example .env

# 编辑 .env 文件，填入你的 API Key
nano .env
```

**国内用户**：建议启用镜像源加速，在 `.env` 中配置：
```bash
DOCKER_REGISTRY=docker.1ms.run/
NPM_REGISTRY=https://registry.npmmirror.com/
GO_PROXY=https://goproxy.cn,direct
```

### 2. 启动服务

```bash
# 构建并启动（必须使用 docker compose）
docker compose -p banana-pro up -d

# 查看日志
docker compose -p banana-pro logs -f

# 查看运行状态
docker compose -p banana-pro ps
```

> ⚠️ **注意**：必须使用 `docker compose` 命令，才能读取 `.env` 文件中的配置。

### 3. 访问应用

浏览器打开：http://localhost:8080

---

## 配置说明

### 环境变量

| 变量名 | 说明 | 默认值 |
|-------|------|-------|
| **镜像源配置** | | |
| `DOCKER_REGISTRY` | Docker Hub 镜像源 | 空（官方源） |
| `NPM_REGISTRY` | npm 镜像源 | 空（官方源） |
| `GO_PROXY` | Go 模块代理 | `https://goproxy.cn,direct` |
| **API 配置** | | |
| `GEMINI_API_KEY` | Gemini API 密钥 | - |
| `GEMINI_API_BASE` | Gemini API 地址 | `https://generativelanguage.googleapis.com` |
| `OPENAI_API_KEY` | OpenAI API 密钥 | - |
| `OPENAI_API_BASE` | OpenAI API 地址 | `https://api.openai.com/v1` |
| **服务器配置** | | |
| `SERVER_HOST` | 后端监听地址 | `0.0.0.0` |
| `SERVER_PORT` | 后端监听端口 | `8080` |
| `TZ` | 时区 | `Asia/Shanghai` |

### 国内镜像源推荐

如果遇到 Docker 拉取镜像慢的问题，可以配置以下镜像源：

```bash
# .env 文件
DOCKER_REGISTRY=docker.1ms.run/
NPM_REGISTRY=https://registry.npmmirror.com/
GO_PROXY=https://goproxy.cn,direct
```

**可选的 Docker 镜像源**：
- `docker.1ms.run/` - 稳定快速
- `dockerpull.org/` - 备用源
- `dockerhub.icu/` - 备用源

### 数据持久化

以下目录会自动挂载到宿主机：

- `./data/storage` - 图片存储和数据库

### 自定义配置

如需修改其他配置，可以创建 `config.yaml` 文件：

```bash
# 从模板复制（可选，已有默认配置）
cp backend/configs/config.yaml ./config.yaml

# 编辑配置
nano config.yaml
```

然后重启服务：

```bash
docker compose -p banana-pro restart
```

---

## 常用命令

```bash
# 启动服务
docker compose -p banana-pro up -d

# 停止服务
docker compose -p banana-pro down

# 重启服务
docker compose -p banana-pro restart

# 查看日志
docker compose -p banana-pro logs -f

# 进入容器
docker compose -p banana-pro exec banana-pro sh

# 重新构建镜像
docker compose -p banana-pro up -d --build

# 强制重新构建（无缓存）
docker compose -p banana-pro build --no-cache
docker compose -p banana-pro up -d

# 清理数据（危险操作！）
docker compose -p banana-pro down -v
rm -rf ./data/storage
```

---

## 国内用户加速指南

### 为什么构建很慢？

Docker 构建需要下载：
1. **基础镜像**（node、golang、alpine）
2. **npm 依赖包**（前端）
3. **Go 模块**（后端）

### 解决方案

在 `.env` 文件中配置镜像源：

```bash
# Docker Hub 镜像源（注意末尾斜杠）
DOCKER_REGISTRY=docker.1ms.run/

# npm 淘宝镜像
NPM_REGISTRY=https://registry.npmmirror.com/

# Go 模块代理
GO_PROXY=https://goproxy.cn,direct
```

### 验证镜像源是否生效

```bash
# 查看构建日志
docker compose -p banana-pro build --progress=plain 2>&1 | grep -i "pulling\|downloading"
```

如果看到从 `docker.1ms.run` 拉取镜像，说明配置成功。

---

## 生产环境部署

### 使用 Nginx 反向代理（推荐）

```bash
# 启动带 Nginx 的完整栈
docker compose -p banana-pro --profile production up -d
```

这将启动：
- `banana-pro` - 主应用服务
- `nginx` - Nginx 反向代理（80/443 端口）

### 配置 HTTPS

1. 将 SSL 证书放入 `docker/ssl/` 目录：
   ```
   docker/ssl/cert.pem
   docker/ssl/key.pem
   ```

2. 修改 `docker/nginx-proxy.conf`，启用 HTTPS 配置

3. 重启服务：
   ```bash
   docker compose -p banana-pro --profile production up -d --build
   ```

---

## 故障排查

### 端口被占用

如果 8080 端口已被占用，可以修改映射端口：

```yaml
# docker-compose.yml
ports:
  - "8888:80"  # 使用 8888 端口
```

### API Key 无效

检查环境变量是否正确配置：

```bash
docker compose -p banana-pro exec banana-pro env | grep API
```

### 数据存储问题

检查存储目录权限：

```bash
ls -la ./data/storage
```

确保目录存在且有写权限：

```bash
mkdir -p ./data/storage
chmod 755 ./data/storage
```

### 镜像拉取失败

如果从官方源拉取失败，请启用镜像源配置：

```bash
# 编辑 .env 文件
nano .env

# 取消以下行的注释（或添加）
DOCKER_REGISTRY=docker.1ms.run/
NPM_REGISTRY=https://registry.npmmirror.com/
GO_PROXY=https://goproxy.cn,direct

# 重新构建
docker compose -p banana-pro build --no-cache
```

### 查看详细日志

```bash
# 查看所有日志
docker compose -p banana-pro logs

# 实时跟踪日志
docker compose -p banana-pro logs -f --tail=100

# 只查看后端日志
docker compose -p banana-pro logs -f banana-pro | grep server
```

---

## 推荐 API 服务商

| 服务商 | 特点 | 价格 | 官网 |
|-------|------|------|------|
| [Smart tokenAPI](https://www.smarttoken.cloud) | 兼容 OpenAI 格式，稳定可靠 | 1K 图片 ¥0.08 | [yunwu.ai](https://yunwu.ai) |
| Google Gemini | 官方 API | 1K 图片 ≈ ¥0.94 | [ai.google.dev](https://ai.google.dev) |

---

## 架构说明

```
┌─────────────────────────────────────────┐
│             Docker 容器                  │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │         Nginx (端口 80)         │   │
│  │  - / → 前端静态资源              │   │
│  │  - /api → 后端 API              │   │
│  │  - /storage → 图片存储          │   │
│  └─────────────────────────────────┘   │
│                ↓                        │
│  ┌─────────────────────────────────┐   │
│  │    Go Backend (端口 8080)       │   │
│  │    - /api/v1/*                  │   │
│  │    - /storage/*                 │   │
│  └─────────────────────────────────┘   │
│                                         │
│  /app/storage (数据持久化)              │
└─────────────────────────────────────────┘
```

---

## 相关文件

- `Dockerfile` - 镜像构建文件（支持 Build Args 配置镜像源）
- `docker-compose.yml` - 服务编排配置
- `docker/nginx.conf` - Nginx 配置
- `config.yaml` - 后端配置文件（可选）
- `.env.example` - 环境变量模板
- `.dockerignore` - Docker 构建排除文件
