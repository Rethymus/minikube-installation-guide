# How to Install Minikube from Scratch

English | [简体中文](README-cn.md)

This guide provides step-by-step instructions for installing Minikube on Ubuntu Linux from scratch, including all necessary dependencies.

## Prerequisites

- Ubuntu Linux system
- Internet connection
- sudo privileges

## Installation Steps

### 1. Update System and Install curl

```bash
sudo apt update && sudo apt install -y curl
```

### 2. Install Docker

#### Set up Docker's apt repository

```bash
# Add Docker's official GPG key
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

#### Install Docker packages

```bash
# Install Docker packages (latest version)
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Verify installation
docker --version

# Check Docker service status
sudo systemctl status docker
```

#### Configure Docker user permissions

```bash
# Create docker group (if it doesn't exist)
sudo groupadd docker

# Add current user to docker group
sudo usermod -aG docker $USER

# Apply group changes (logout and login again, or use newgrp)
newgrp docker
```

### 3. Configure Docker Storage

```bash
# Check current Docker storage information
docker info | grep "Storage Driver\|Disk"

# Clean up unused containers and images (free disk space)
docker system prune -a --volumes

# Set Docker default storage path (avoid disk space issues)
sudo mkdir -p /mnt/data/docker
sudo sed -i '/"data-root"/d' /etc/docker/daemon.json
echo '{"data-root": "/mnt/data/docker"}' | sudo tee /etc/docker/daemon.json

# Restart Docker service
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 4. Install Portainer (Optional)

Portainer provides a web-based Docker management interface.

```bash
# Create persistent data volume
docker volume create portainer_data

# Run Portainer container
docker run -d -p 9000:9000 --name portainer --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce
```

Access Portainer at `http://localhost:9000`

### 5. Install Minikube

#### Install kubectl

```bash
# Install kubectl (fix exec format error)
curl -LO "https://dl.k8s.io/release/$(curl -LS https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl && sudo mv kubectl /usr/local/bin/
```

#### Install Minikube

```bash
# Install Minikube (using Chinese mirror for faster download)
curl -Lo minikube https://mirrors.tuna.tsinghua.edu.cn/github-release/kubeadm/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube && sudo mv minikube /usr/local/bin/
```

#### Start Minikube

```bash
# Start Minikube (specify Chinese image repository for faster setup)
minikube start \
  --driver=docker \
  --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers \
  --binary-mirror=https://mirrors.tuna.tsinghua.edu.cn/kubernetes
```

### 6. Verification

```bash
# Verify Docker status
docker ps | grep portainer

# Verify Minikube status
minikube status

# Verify Kubernetes components
kubectl get nodes
```

## Cleanup Commands

If you need to clean up your Kubernetes resources:

```bash
# View all deployments
kubectl get deployments

# Delete all non-Minikube deployments
kubectl delete deployment $(kubectl get deployments -o jsonpath='{.items[*].metadata.name}')

# View all services
kubectl get services

# Delete all non-system services
kubectl delete service $(kubectl get services -o jsonpath='{.items[*].metadata.name}' | grep -v 'kubernetes')

# View pods (check for remaining pods)
kubectl get pods

# Delete remaining pods if any
kubectl delete pod $(kubectl get pods -o jsonpath='{.items[*].metadata.name}')
```

## Troubleshooting

- If you encounter permission issues with Docker, make sure you've logged out and back in after adding your user to the docker group
- If Minikube fails to start, try `minikube delete` and then start again
- For storage issues, ensure you have enough disk space in `/mnt/data/docker`

## Next Steps

After installation, you can:
- Deploy applications to your Minikube cluster
- Use `kubectl` to manage your Kubernetes resources
- Access the Kubernetes dashboard with `minikube dashboard`
- Use Portainer to manage Docker containers via web interface

## Contributing

Feel free to submit issues and enhancement requests!