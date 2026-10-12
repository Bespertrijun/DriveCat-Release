# DriveCat

自部署的网盘同步与管理工具。**闭源软件**，本仓库仅提供安装说明与更新日志，不包含源代码。

## 功能特性

- **多网盘支持**：115 网盘、Google Drive（持续增加中）
- **目录监控同步**：监控本地目录，文件变动自动同步到网盘
- **传输任务管理**：上传 / 下载任务队列，支持断点续传、批量操作
- **文件浏览器**：类 Google Drive 的网页文件管理界面，支持拖拽、搜索、移动端适配
- **定时任务**：计划执行同步任务
- **插件系统**：可扩展的插件机制
- **在线自更新**：应用内一键升级（可选）

## 快速安装（Docker Compose）

前置要求：已安装 Docker 与 Docker Compose。

1. 新建一个目录，把本仓库的 [`docker-compose.yml`](./docker-compose.yml) 放进去；

2. 按需修改 compose 文件中的监控目录挂载：

   ```yaml
   volumes:
     - /path/to/watch:/watch       # ← 改成你要监控同步的目录
   ```

3. 启动：

   ```bash
   docker compose up -d
   ```

4. 浏览器打开 `http://<服务器IP>:9100`，首次启动会进入 **Web 初始化向导**，按提示创建管理员账号即可。

## 配置说明

### 端口

| 端口 | 说明 |
| --- | --- |
| `9100` | Web 界面与 API（必需） |
| `8443` | Google OAuth HTTPS 回调（仅远程部署且启用 SSL 时需要） |

### 数据持久化

`./data` 挂载到容器内 `/app/backend/data`，存放数据库、日志和插件数据。**备份此目录即可完整备份 DriveCat。**

### 监控目录挂载

需要同步哪个本地目录，就把它挂载进容器，格式为 `宿主机路径:容器内路径`，可挂载多个：

```yaml
volumes:
  - /mnt/media:/watch/media
  - /mnt/documents:/watch/documents
```

然后在 Web 界面中添加监控任务时选择对应的容器内路径。

### Docker socket（可选）

```yaml
- /var/run/docker.sock:/var/run/docker.sock
```

挂载后 DriveCat 可使用**应用内在线更新**功能。该配置赋予容器对宿主 Docker 的完整控制权，如不需要自更新，注释掉此行以减小攻击面，改用手动更新（见下文）。

## 升级

### 应用内自更新

挂载了 docker.sock 时，可直接在 Web 界面中检查更新并升级。

### 手动升级

```bash
docker compose pull
docker compose up -d
```

## Google Drive 授权说明

Google 对已发布的 OAuth 应用强制要求 HTTPS 重定向地址。本地 / Docker 部署通常没有 HTTPS，DriveCat 提供了一个共享的**官方回调端点**：

```
https://oauth.hwii.de/api/drives/google/callback
```

在网盘页面点击「官方回调 → 授权连接」即可完成授权。该端点只转发授权码，**不持有、不存储**任何 Google 凭据，Token 交换始终由你自己的 DriveCat 后端完成。

也可以在自己的 Google Cloud Console 创建 OAuth 客户端，使用自有回调地址。

## 更新日志

见 [CHANGELOG.md](./CHANGELOG.md)。

## 系统要求

- Docker 20.10+
- 内存：建议 512MB 以上
- 架构：linux/amd64、linux/arm64

## 反馈与授权

- 问题反馈：在本仓库提交 [Issue](../../issues)
- 本软件为闭源软件，镜像发布于 [Docker Hub](https://hub.docker.com/r/kawaii0v0/drivecat)
