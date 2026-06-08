# Docker详解

## Docker是什么？
&emsp;&emsp;Docker是一个开源的容器化平台，允许开发者将应用程序及其依赖打包成一个轻量级、可移植的容器。Docker容器可以在任何支持Docker的环境中运行，实现了应用程序的快速部署和跨平台运行。

## Docker的特点

| 特性 | 说明 |
|------|------|
| **轻量级** | 容器共享宿主机内核，启动速度快 |
| **可移植** | 一次构建，处处运行 |
| **隔离性** | 资源隔离，互不干扰 |
| **可扩展** | 支持容器编排和集群管理 |
| **标准化** | 统一的打包和部署格式 |

---

## 一、Docker核心原理

### 1.1 Docker架构

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         Docker Host                                   │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐            │
│  │   Docker     │    │   Docker     │    │   Docker     │            │
│  │   Client     │    │   Daemon     │    │   Registry   │            │
│  │  (命令行)    │    │  (守护进程)   │    │  (镜像仓库)   │            │
│  └──────┬───────┘    └──────┬───────┘    └──────┬───────┘            │
│         │                   │                   │                     │
│         │ REST API          │                   │                     │
│         ▼                   ▼                   ▼                     │
│  ┌──────────────────────────────────────────────────────────────┐    │
│  │                      Container Runtime                        │    │
│  │  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    │    │
│  │  │    RunC      │    │   containerd │    │    OCI       │    │    │
│  │  │ (运行时)     │    │ (容器管理)   │    │  (标准规范)   │    │    │
│  │  └──────────────┘    └──────────────┘    └──────────────┘    │    │
│  └──────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Docker组件

**Docker Client**：命令行工具，用于与Docker Daemon通信

**Docker Daemon**：后台守护进程，管理容器的创建、运行和销毁

**Docker Registry**：镜像仓库，存储Docker镜像

**Docker Image**：只读模板，包含应用程序及其依赖

**Docker Container**：运行中的实例，由镜像创建

### 1.3 Docker镜像原理

**镜像分层结构**：
```
┌─────────────────────────────────────────────────────┐
│  Layer 3: 应用程序代码                              │
│  (RUN npm install)                                  │
├─────────────────────────────────────────────────────┤
│  Layer 2: 依赖安装                                  │
│  (COPY package.json)                                │
├─────────────────────────────────────────────────────┤
│  Layer 1: 基础镜像                                  │
│  (FROM node:18-alpine)                             │
├─────────────────────────────────────────────────────┤
│  Layer 0: 操作系统层                                │
│  (alpine:3.18)                                     │
└─────────────────────────────────────────────────────┘
```

**Dockerfile示例**：
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install --production
COPY . .
EXPOSE 3000
CMD ["node", "server.js"]
```

### 1.4 Docker容器原理

**容器与虚拟机的区别**：

| 特性 | Docker容器 | 虚拟机 |
|------|-----------|--------|
| 启动时间 | 秒级 | 分钟级 |
| 资源占用 | 低 | 高 |
| 隔离性 | 进程级 | 系统级 |
| 性能 | 接近原生 | 有开销 |

**容器创建流程**：
```
docker run → 拉取镜像 → 创建容器 → 启动进程 → 运行容器
```

### 1.5 Docker网络

**网络模式**：

| 模式 | 说明 |
|------|------|
| **bridge** | 默认模式，容器间通过网桥通信 |
| **host** | 共享宿主机网络 |
| **none** | 无网络 |
| **container** | 共享其他容器网络 |

**网络配置示例**：
```bash
# 创建自定义网络
docker network create --driver bridge my-network

# 连接容器到网络
docker run --network my-network --name container1 nginx
```

---

## 二、Docker常见问题分析与排查

### 问题1：Docker无法启动

**现象**：
```
docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

**排查步骤**：
```bash
# 1. 检查Docker服务状态
systemctl status docker

# 2. 启动Docker服务
systemctl start docker

# 3. 检查Docker socket权限
ls -la /var/run/docker.sock

# 4. 检查日志
journalctl -u docker
```

**解决方案**：
1. 启动Docker服务
2. 添加用户到docker组
3. 检查Docker配置

---

### 问题2：容器无法访问外部网络

**现象**：
- 容器内无法ping通外部地址
- 网络请求超时

**排查步骤**：
```bash
# 1. 检查容器网络配置
docker inspect container_name | grep -A 20 "NetworkSettings"

# 2. 测试容器内网络
docker exec -it container_name ping 8.8.8.8

# 3. 检查宿主机网络
ping 8.8.8.8

# 4. 检查防火墙规则
iptables -L
```

**解决方案**：
1. 检查DNS配置
2. 检查防火墙规则
3. 重启Docker网络

---

### 问题3：容器启动失败

**现象**：
```
docker: Error response from daemon: OCI runtime create failed...
```

**排查步骤**：
```bash
# 1. 查看容器日志
docker logs container_name

# 2. 检查镜像是否存在
docker images

# 3. 检查容器配置
docker inspect container_name

# 4. 手动运行容器测试
docker run -it --rm image_name /bin/bash
```

**解决方案**：
1. 检查Dockerfile配置
2. 检查镜像完整性
3. 检查资源限制

---

### 问题4：镜像拉取失败

**现象**：
```
Error response from daemon: pull access denied for xxx, repository does not exist or may require 'docker login'
```

**排查步骤**：
```bash
# 1. 检查网络连接
curl https://hub.docker.com

# 2. 检查镜像名称
docker search image_name

# 3. 登录Docker Hub
docker login

# 4. 检查镜像标签
docker pull image_name:tag
```

**解决方案**：
1. 登录Docker Hub
2. 检查镜像名称和标签
3. 配置镜像加速器

---

### 问题5：容器资源占用过高

**现象**：
- CPU使用率高
- 内存占用高
- 磁盘空间不足

**排查步骤**：
```bash
# 1. 查看容器资源使用
docker stats

# 2. 查看磁盘使用
docker system df

# 3. 清理无用资源
docker system prune

# 4. 检查容器日志大小
docker logs --tail 100 container_name
```

**解决方案**：
1. 配置资源限制
2. 清理无用镜像和容器
3. 优化应用程序

---

### 问题6：Docker网络冲突

**现象**：
```
Error response from daemon: could not find an available, non-overlapping IPv4 address pool among the defaults to assign to the network
```

**排查步骤**：
```bash
# 1. 检查现有网络
docker network ls

# 2. 检查网络配置
docker network inspect bridge

# 3. 查看宿主机网络
ip addr show
```

**解决方案**：
1. 创建自定义网络时指定子网
2. 修改默认网络配置
3. 重启Docker服务

---

### 问题7：容器数据丢失

**现象**：
- 容器重启后数据丢失
- 挂载的数据卷不可用

**排查步骤**：
```bash
# 1. 检查数据卷配置
docker volume ls

# 2. 检查容器挂载
docker inspect container_name | grep -A 10 "Mounts"

# 3. 检查数据卷路径
ls -la /var/lib/docker/volumes/
```

**解决方案**：
1. 使用数据卷持久化数据
2. 检查挂载路径权限
3. 备份重要数据

---

### 问题8：Docker Compose启动失败

**现象**：
```
docker-compose up: ERROR: Service 'xxx' failed to build
```

**排查步骤**：
```bash
# 1. 检查docker-compose.yml
docker-compose config

# 2. 查看构建日志
docker-compose build --no-cache

# 3. 检查依赖服务
docker-compose ps
```

**解决方案**：
1. 修复docker-compose.yml配置
2. 检查依赖服务状态
3. 清理构建缓存

---

### 问题9：容器间通信失败

**现象**：
- 容器间无法通过容器名通信
- DNS解析失败

**排查步骤**：
```bash
# 1. 检查网络配置
docker network inspect network_name

# 2. 测试容器间通信
docker exec -it container1 ping container2

# 3. 检查DNS配置
docker exec -it container1 cat /etc/resolv.conf
```

**解决方案**：
1. 使用自定义网络
2. 配置容器别名
3. 检查DNS服务

---

### 问题10：Docker权限问题

**现象**：
```
Got permission denied while trying to connect to the Docker daemon socket
```

**排查步骤**：
```bash
# 1. 检查Docker socket权限
ls -la /var/run/docker.sock

# 2. 检查用户组
groups $USER

# 3. 检查SELinux状态
getenforce
```

**解决方案**：
1. 添加用户到docker组
2. 修改Docker socket权限
3. 关闭SELinux或配置规则

---

## 三、Docker面试题

### 基础问题

**Q1：Docker是什么？有什么特点？**

**A1：**
Docker是一个容器化平台，具有以下特点：
- 轻量级：容器共享宿主机内核
- 可移植：一次构建，处处运行
- 隔离性：资源隔离，互不干扰
- 可扩展：支持容器编排

---

**Q2：Docker容器和虚拟机有什么区别？**

**A2：**

| 特性 | Docker容器 | 虚拟机 |
|------|-----------|--------|
| 启动时间 | 秒级 | 分钟级 |
| 资源占用 | 低 | 高 |
| 隔离性 | 进程级 | 系统级 |
| 性能 | 接近原生 | 有开销 |
| 镜像大小 | 小 | 大 |

---

**Q3：Docker的核心组件有哪些？**

**A3：**
- **Docker Client**：命令行工具
- **Docker Daemon**：后台守护进程
- **Docker Registry**：镜像仓库
- **Docker Image**：只读模板
- **Docker Container**：运行实例

---

**Q4：Docker镜像的分层结构是什么？**

**A4：**
Docker镜像采用分层存储，每层都是只读的：
- 底层是基础镜像（如alpine、ubuntu）
- 上层是应用代码和依赖
- 层可以被多个镜像共享，节省空间

---

**Q5：Dockerfile的常用指令有哪些？**

**A5：**
- `FROM`：指定基础镜像
- `RUN`：执行命令
- `COPY`：复制文件
- `ADD`：复制文件（支持URL和自动解压）
- `WORKDIR`：设置工作目录
- `EXPOSE`：暴露端口
- `CMD`：容器启动命令
- `ENTRYPOINT`：容器入口点

---

**Q6：Docker的网络模式有哪些？**

**A6：**
- **bridge**：默认模式，容器间通过网桥通信
- **host**：共享宿主机网络
- **none**：无网络
- **container**：共享其他容器网络

---

**Q7：什么是Docker数据卷？**

**A7：**
数据卷是用于持久化容器数据的存储方式：
- 数据卷可以在容器间共享
- 数据卷独立于容器生命周期
- 支持本地目录挂载和命名卷

---

**Q8：Docker Compose是什么？**

**A8：**
Docker Compose是用于定义和运行多容器应用的工具：
- 使用YAML文件定义服务
- 一键启动多个容器
- 支持网络和卷的自动配置

---

**Q9：如何优化Docker镜像大小？**

**A9：**
- 使用多阶段构建
- 使用更小的基础镜像（如alpine）
- 清理构建缓存和无用文件
- 合并RUN命令

---

**Q10：Docker的安全注意事项有哪些？**

**A10：**
- 不要以root用户运行容器
- 使用最新版本的镜像
- 限制容器资源
- 不要暴露敏感端口

---

### 复杂场景问题

**Q11：如何设计一个高可用的Docker集群？**

**A11：**

**架构设计：**
```
                    ┌─────────────────────────────┐
                    │         负载均衡器           │
                    │     (Nginx/HAProxy)        │
                    └─────────────┬───────────────┘
                                  │
        ┌─────────────────────────┼─────────────────────────┐
        ▼                         ▼                         ▼
┌──────────────┐          ┌──────────────┐          ┌──────────────┐
│   Node 1     │          │   Node 2     │          │   Node 3     │
│  (Manager)   │◄─────────►│  (Manager)   │◄─────────►│  (Worker)    │
│  Docker Swarm│          │  Docker Swarm│          │  Docker Swarm│
└──────────────┘          └──────────────┘          └──────────────┘
        │                         │                         │
        └─────────────────────────┼─────────────────────────┘
                                  │
                    ┌─────────────────────────────────────┐
                    │           共享存储                  │
                    │    (NFS/Ceph/GlusterFS)            │
                    └─────────────────────────────────────┘
```

**关键配置：**
```yaml
version: '3.8'
services:
  web:
    image: nginx
    deploy:
      replicas: 3
      restart_policy:
        condition: on-failure
      update_config:
        parallelism: 1
        delay: 10s
    networks:
      - webnet
networks:
  webnet:
    driver: overlay
```

**监控告警：**
- 监控节点状态
- 监控容器健康
- 监控资源使用

---

**Q12：如何实现Docker镜像的安全扫描？**

**A12：**

**方案设计：**

1. **使用Clair扫描镜像**：
```bash
# 安装Clair
docker run -d --name clair postgres
docker run -d --name clair_cli arminc/clair-cli

# 扫描镜像
clair-scanner --ip <clair-ip> my-image:latest
```

2. **使用Trivy扫描**：
```bash
# 安装Trivy
brew install aquasecurity/trivy/trivy

# 扫描镜像
trivy image my-image:latest
```

3. **集成到CI/CD流程**：
```yaml
# .gitlab-ci.yml
scan:
  stage: test
  image: aquasec/trivy
  script:
    - trivy image --exit-code 1 --severity HIGH,CRITICAL my-image:latest
```

---

**Q13：如何处理Docker容器的日志管理？**

**A13：**

**日志驱动配置：**
```dockerfile
# docker-compose.yml
services:
  web:
    image: nginx
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

**日志收集方案：**
```
容器日志 → Fluentd → Elasticsearch → Kibana
```

**配置示例：**
```yaml
# fluentd.conf
<source>
  @type forward
  port 24224
</source>

<match *.**>
  @type elasticsearch
  host elasticsearch
  port 9200
  index_name docker-log
</match>
```

---

**Q14：如何实现Docker容器的自动扩缩容？**

**A14：**

**方案一：Docker Swarm自动扩缩容**
```yaml
version: '3.8'
services:
  web:
    image: nginx
    deploy:
      mode: replicated
      replicas: 3
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
```

**方案二：使用Prometheus + Alertmanager + Scale**
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'docker'
    static_configs:
      - targets: ['localhost:9323']
```

**方案三：使用Kubernetes Horizontal Pod Autoscaler**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web
  minReplicas: 3
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

---

**Q15：如何进行Docker镜像的版本管理？**

**A15：**

**版本策略：**
```
my-image:latest      # 最新稳定版
my-image:1.0         # 主版本
my-image:1.0.0       # 完整版本
my-image:1.0.0-rc1   # 候选版本
my-image:sha256:xxx  # 摘要版本
```

**镜像标签管理：**
```bash
# 构建并打标签
docker build -t my-image:1.0.0 .
docker tag my-image:1.0.0 my-image:latest
docker tag my-image:1.0.0 my-image:1.0

# 推送镜像
docker push my-image:1.0.0
docker push my-image:latest
docker push my-image:1.0
```

**镜像清理策略：**
```bash
# 清理旧镜像
docker image prune -a --filter "until=24h"

# 清理未使用的资源
docker system prune -f
```

---

**Q16：如何实现Docker容器的安全隔离？**

**Q16：**

**方案设计：**

1. **使用用户命名空间**：
```bash
# /etc/docker/daemon.json
{
  "userns-remap": "default"
}
```

2. **限制容器能力**：
```dockerfile
# docker-compose.yml
services:
  web:
    image: nginx
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
```

3. **使用安全扫描**：
```bash
trivy image --exit-code 1 my-image:latest
```

4. **配置SELinux**：
```bash
docker run --security-opt label=type:container_t nginx
```

---

**Q17：如何处理Docker的存储问题？**

**A17：**

**存储方案对比：**

| 方案 | 优点 | 缺点 |
|------|------|------|
| **local** | 简单、快速 | 不支持集群 |
| **nfs** | 共享存储 | 性能有限 |
| **ceph** | 分布式、高可用 | 复杂 |
| **glusterfs** | 分布式、弹性 | 性能一般 |

**配置示例：**
```yaml
version: '3.8'
services:
  web:
    image: nginx
    volumes:
      - data:/var/www/html
volumes:
  data:
    driver: local
    driver_opts:
      type: nfs
      o: addr=nfs-server,rw
      device: ":/path/to/share"
```

---

**Q18：如何实现Docker容器的网络隔离？**

**A18：**

**网络隔离策略：**

1. **使用自定义网络**：
```bash
# 创建隔离网络
docker network create --internal --driver bridge isolated-net

# 连接容器到隔离网络
docker run --network isolated-net --name secure-app nginx
```

2. **配置网络规则**：
```bash
# 禁止容器访问外部网络
docker network create --internal restricted-net

# 允许特定端口访问
iptables -A DOCKER -i docker0 -p tcp --dport 80 -j ACCEPT
```

3. **使用Docker Compose网络**：
```yaml
version: '3.8'
services:
  web:
    image: nginx
    networks:
      - frontend
  db:
    image: mysql
    networks:
      - backend
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true
```

---

**Q19：如何进行Docker容器的备份和恢复？**

**A19：**

**备份策略：**

1. **备份容器数据**：
```bash
# 导出容器文件系统
docker export container_name > container.tar

# 备份数据卷
docker run --rm -v my-volume:/data -v /backup:/backup busybox tar cvf /backup/volume.tar /data
```

2. **备份镜像**：
```bash
# 保存镜像
docker save my-image:latest > my-image.tar

# 加载镜像
docker load < my-image.tar
```

3. **自动化备份脚本**：
```bash
#!/bin/bash
DATE=$(date +%Y%m%d)
docker run --rm -v data:/data -v /backup:/backup busybox tar cvzf /backup/data-$DATE.tar.gz /data
```

---

**Q20：如何实现Docker的持续集成和持续部署？**

**A20：**

**CI/CD流程：**
```
代码提交 → 构建镜像 → 安全扫描 → 部署测试 → 部署生产
```

**GitLab CI配置示例：**
```yaml
stages:
  - build
  - test
  - deploy

build:
  stage: build
  script:
    - docker build -t my-image:$CI_COMMIT_SHA .
    - docker push my-image:$CI_COMMIT_SHA

test:
  stage: test
  script:
    - docker run my-image:$CI_COMMIT_SHA npm test
    - trivy image --exit-code 1 my-image:$CI_COMMIT_SHA

deploy:
  stage: deploy
  script:
    - docker pull my-image:$CI_COMMIT_SHA
    - docker tag my-image:$CI_COMMIT_SHA my-image:latest
    - docker service update --image my-image:latest web-service
```

<center>...未完待续...</center>
---  
***  

<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.css">
<script src="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.min.js"></script>
<div id="gitalk-container"></div>
<script>
  var gitalk = new Gitalk({
    "clientID": "44d7c96f948be236a8c9",
    "clientSecret": "fb9fb3178db6640131c4e3eb69f9449e42bba661",
    "repo": "blog",
    "owner": "Zhang-jie-jun",
    "admin": ["Zhang-jie-jun"],
    "id": location.pathname,      
    "distractionFreeMode": false  
  });
  gitalk.render("gitalk-container");
</script>
