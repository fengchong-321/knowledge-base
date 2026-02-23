# Docker 常用命令速查

> 标签: #docker #容器 #devops
> 创建时间: 2026-02-23
> 来源: [阿里云速查手册](https://developer.aliyun.com/article/1702784) | [掘金实战指南](https://juejin.cn/post/7591799107486859274)

## 概述

Docker 常用命令速查表，覆盖镜像管理、容器操作、网络、数据卷、Compose 等核心场景。

## 镜像管理

```bash
# 搜索镜像
docker search nginx

# 拉取镜像
docker pull nginx:latest
docker pull nginx:1.24

# 查看本地镜像
docker images
docker images -a

# 删除镜像
docker rmi nginx:latest
docker rmi -f $(docker images -q)  # 删除所有镜像

# 构建镜像
docker build -t myapp:v1 .
docker build -t myapp:v1 -f Dockerfile.prod .

# 导出/导入镜像
docker save -o nginx.tar nginx:latest
docker load -i nginx.tar
```

## 容器生命周期

```bash
# 运行容器
docker run -d -p 80:80 --name web nginx
docker run -it ubuntu /bin/bash  # 交互式

# 常用参数
-d          # 后台运行
-p 80:80    # 端口映射
-v /host:/container  # 挂载卷
-e KEY=VAL  # 环境变量
--restart always  # 自动重启

# 启动/停止/重启
docker start web
docker stop web
docker restart web
docker kill web  # 强制停止

# 删除容器
docker rm web
docker rm -f $(docker ps -aq)  # 删除所有容器
```

## 容器监控与调试

```bash
# 查看运行中的容器
docker ps
docker ps -a  # 包含已停止

# 查看日志
docker logs web
docker logs -f --tail 100 web  # 实时跟踪

# 进入容器
docker exec -it web /bin/bash
docker exec -it web sh  # Alpine

# 查看资源占用
docker stats
docker stats web

# 查看进程
docker top web

# 查看详情
docker inspect web
```

## 网络管理

```bash
# 创建网络
docker network create mynet
docker network create --driver bridge mynet

# 查看网络
docker network ls

# 连接容器到网络
docker network connect mynet web

# 断开网络
docker network disconnect mynet web

# 删除网络
docker network rm mynet
```

## 数据卷

```bash
# 创建卷
docker volume create mydata

# 查看卷
docker volume ls

# 查看卷详情
docker volume inspect mydata

# 删除卷
docker volume rm mydata

# 使用卷挂载
docker run -v mydata:/app/data nginx
docker run -v /host/path:/container/path nginx
```

## Docker Compose

```bash
# 启动服务
docker-compose up -d

# 停止服务
docker-compose down

# 查看日志
docker-compose logs -f

# 重启服务
docker-compose restart

# 重新构建
docker-compose up -d --build

# 查看状态
docker-compose ps
```

## 系统维护

```bash
# 查看磁盘占用
docker system df

# 清理未使用资源
docker system prune        # 清理所有未使用资源
docker system prune -a     # 连镜像一起清理
docker volume prune        # 清理未使用卷
docker image prune         # 清理悬空镜像

# 查看版本信息
docker version
docker info
```

## 快速参考表

| 操作 | 命令 |
|------|------|
| 运行容器 | `docker run -d --name web -p 80:80 nginx` |
| 查看容器 | `docker ps -a` |
| 查看日志 | `docker logs -f web` |
| 进入容器 | `docker exec -it web bash` |
| 停止容器 | `docker stop web` |
| 删除容器 | `docker rm -f web` |
| 清理系统 | `docker system prune -a` |

## 相关知识点

- [[Docker Compose 配置详解]]
- [[Kubernetes 基础概念]]

---
*采集自 Claude Code 对话*
