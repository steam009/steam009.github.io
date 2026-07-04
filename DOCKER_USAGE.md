# 使用 Docker Compose 构建和运行网站

这是一个基于 Jekyll 的学术网站，可以使用 Docker Compose 轻松构建和运行。

## 前置要求

- 安装 [Docker Desktop](https://www.docker.com/products/docker-desktop/) (Windows/Mac) 或 Docker Engine (Linux)
- 安装 Docker Compose (通常包含在 Docker Desktop 中)

## 快速开始

### 1. 构建并启动服务

在项目根目录下运行：

```bash
docker-compose up --build
```

这个命令会：
- 构建 Docker 镜像（首次运行或 Dockerfile 有变化时）
- 启动 Jekyll 开发服务器
- 在后台运行并显示日志

### 2. 访问网站

打开浏览器访问：http://localhost:4000

### 3. 后台运行

如果想让服务在后台运行（不占用终端），使用：

```bash
docker-compose up -d --build
```

### 4. 查看日志

如果服务在后台运行，查看日志使用：

```bash
docker-compose logs -f
```

### 5. 停止服务

停止运行中的服务：

```bash
docker-compose down
```

## 常用命令

### 重新构建镜像

如果修改了 `Dockerfile` 或 `Gemfile`，需要重新构建：

```bash
docker-compose build --no-cache
docker-compose up
```

### 清理所有数据

停止服务并删除容器、网络和卷：

```bash
docker-compose down -v
```

### 进入容器

如果需要进入容器内部执行命令：

```bash
docker-compose exec jekyll-site bash
```

### 查看运行状态

```bash
docker-compose ps
```

## 配置说明

- **端口**: 网站运行在 `4000` 端口
- **配置文件**: 使用 `_config.yml` 和 `_config.local.yml` 两个配置文件
- **热重载**: 修改源文件后，Jekyll 会自动重新生成网站（watch 模式）
- **卷挂载**: 项目目录挂载到容器中，修改会实时反映

## 故障排除

### 端口被占用

如果 4000 端口被占用，可以修改 `docker-compose.yaml` 中的端口映射：

```yaml
ports:
  - "4001:4000"  # 将本地端口改为 4001
```

### 权限问题

如果遇到文件权限问题，可能需要调整 Dockerfile 中的用户设置。

### 清理缓存

如果遇到依赖问题，可以清理 bundle 缓存：

```bash
docker-compose down -v
docker-compose build --no-cache
docker-compose up
```

## 开发工作流

1. 启动服务：`docker-compose up`
2. 编辑文件（Markdown、HTML、CSS 等）
3. Jekyll 会自动检测变化并重新生成
4. 刷新浏览器查看更改
5. 完成后停止服务：`docker-compose down`
