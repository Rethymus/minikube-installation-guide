# 如何从零开始安装 Minikube

[English](readme.md) | 简体中文

本指南提供了在 Ubuntu Linux 系统上从零开始安装 Minikube 的详细步骤，包括所有必要的依赖项。

## 系统要求

- Ubuntu Linux 系统
- 互联网连接
- sudo 权限

## 安装步骤

### 1. 更新系统并安装 curl

```bash
sudo apt update && sudo apt install -y curl
```

### 2. 安装 Docker

在开始安装前，若想先熟悉 Docker 基础信息，建议阅读 [Docker文档](docker.md) 。 

#### 设置 Docker 的 apt 存储库

```bash
# 添加 Docker 官方 GPG 密钥
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# 将存储库添加到 Apt 源
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

#### 安装 Docker 软件包

```bash
# 安装 Docker 软件包（最新版本）
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 验证安装
docker --version

# 验证 Docker 服务状态
sudo systemctl status docker
```

#### 配置 Docker 用户权限

```bash
# 创建 docker 用户组（如果不存在）
sudo groupadd docker

# 将当前用户加入 docker 组
sudo usermod -aG docker $USER

# 使组权限生效（退出终端并重新登录，或使用 newgrp）
newgrp docker
```

### 3. 配置 Docker 存储

```bash
# 查看当前 Docker 存储信息
docker info | grep "Storage Driver\|Disk"

# 清理无用容器和镜像（释放磁盘空间）
docker system prune -a --volumes

# 设置 Docker 默认存储路径（避免磁盘空间不足）
sudo mkdir -p /mnt/data/docker
sudo sed -i '/"data-root"/d' /etc/docker/daemon.json
echo '{"data-root": "/mnt/data/docker"}' | sudo tee /etc/docker/daemon.json

# 重启 Docker 服务
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 4. 安装 Portainer（可选）

Portainer 提供基于 Web 的 Docker 管理界面。

```bash
# 创建持久化数据卷
docker volume create portainer_data

# 运行 Portainer 容器
docker run -d -p 9000:9000 --name portainer --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce
```

通过 `http://localhost:9000` 访问 Portainer

### 5. 安装 Minikube

#### 安装 kubectl

```bash
# 安装 kubectl（修复 exec format error）
curl -LO "https://dl.k8s.io/release/$(curl -LS https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

#### 安装 Minikube

```bash
# 安装 Minikube（使用国内镜像源加速下载）
curl -Lo minikube https://mirrors.tuna.tsinghua.edu.cn/github-release/kubeadm/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube && sudo mv minikube /usr/local/bin/
```

#### 启动 Minikube

```bash
# 启动 Minikube（指定国内镜像仓库加速安装）
minikube start \
  --driver=docker \
  --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers \
  --binary-mirror=https://mirrors.tuna.tsinghua.edu.cn/kubernetes
```

### 6. 验证安装

```bash
# 验证 Docker 状态
docker ps | grep portainer

# 验证 Minikube 状态
minikube status

# 验证 Kubernetes 组件
kubectl get nodes
```

## 清理命令

如果需要清理 Kubernetes 资源：

```bash
# 查看所有 deployment
kubectl get deployments

# 删除所有非 Minikube 的 deployment
kubectl delete deployment $(kubectl get deployments -o jsonpath='{.items[*].metadata.name}')

# 查看所有 service
kubectl get services

# 删除所有非系统自带的 service
kubectl delete service $(kubectl get services -o jsonpath='{.items[*].metadata.name}' | grep -v 'kubernetes')

# 查看 pods（确认是否还有残留）
kubectl get pods

# 如果有残留，也可以一并删除
kubectl delete pod $(kubectl get pods -o jsonpath='{.items[*].metadata.name}')
```

## 故障排除

- 如果遇到 Docker 权限问题，请确保在将用户添加到 docker 组后已经注销并重新登录
- 如果 Minikube 启动失败，可以尝试 `minikube delete` 然后重新启动
- 如果遇到存储问题，请确保 `/mnt/data/docker` 有足够的磁盘空间

## 后续步骤

安装完成后，您可以：
- 将应用程序部署到 Minikube 集群
- 使用 `kubectl` 管理 Kubernetes 资源
- 通过 `minikube dashboard` 访问 Kubernetes 仪表板
- 使用 Portainer 通过 Web 界面管理 Docker 容器

## 贡献

欢迎提交问题和功能请求！
