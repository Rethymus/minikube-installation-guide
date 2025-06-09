# 学 docker

[官方安装](https://docs.docker.com/engine/install/ubuntu/#uninstall-old-versions)

这个 Install using the apt repository

1. 安装 docker(非官方)

```
mkdir ~/docker && cd ~/docker &&
sudo apt-get -y install docker.io
```

2. 安装 docker-compose(非官方)

```
sudo apt install -y docker-compose
```

3. docker 配置代理(有科学上网)

```bash
sudo mkdir -p /etc/systemd/system/docker.service.d/
sudo vim /etc/systemd/system/docker.service.d/http-proxy.conf
```

http-proxy.conf 文件内容如下:

```bash
[Service]
Environment="HTTP_PROXY=127.0.0.1:7890/"  # 不要加http://
Environment="HTTPS_PROXY=127.0.0.1:7890/" # 不要加https://
```

```bash
sudo vim /etc/docker/daemon.json
```

daemon.json 文件内容如下:

```bash
{
 "registry-mirrors": [
    "https://hub.docker.com/"]
}
```

重启 docker

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo docker info # 查看是否生效
```

vscode 插件 docker 无权限

```bash
sudo chmod 666 /var/run/docker.sock
```

4. 添加用户到 docker 组
   1. `sudo usermod -aG docker $USER`
   2. 重新登录用户`newgrp docker`
   3. 测试`docker ps`
5. 更换国内镜像源
6. 查看 docker 镜像

```
sudo docker image ls
```

4. 下载镜像

```
sudo docker pull 名称
```

5. 启动容器

```
sudo docker run --name
```

```bash
2d3abf607f69   91ef0af61f39   "/bin/sh"   4 minutes ago    Exited (7) 2 minutes ago  my_alpine
docker start my_alpine # 容器是退出状态的,这样启动
```

6. 停止容器

```
sudo docker stop 名称/id
```

## 基于我的停车场管理系统学习使用 docker

### 开 3 个 docker 容器

1. mysql
2. redis
3. 停车场管理系统

### 启动容器(都要加 sudo)

#### 启动 mysql 容器

```bash
docker-compose up -d mysql # 启动 mysql 容器
docker-compose ps # 查看容器状态
docker exec -it <name> bash # 进入容器,用容器的名称<name>这个位置
mysql -u root -p # 登录 mysql 用户名是root,密码看yml文件
```

不持久化的 mysql
`docker run --name my-mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.0`

持久化的 mysql
`docker run -d --name heima -e MYSQL_ROOT_PASSWORD=123456 -e MYSQL_DATABASE=hmdp -v /home/cl/my-gocode/hm-dianping/src/main/resources/db/init.sql:/docker-entrypoint-initdb.d/init.sql -v heima:/var/lib/mysql -p 3306:3306 -d mysql:8.0`

mysql 的配置文件在`/etc/my.cnf`

给 mysql 容器设置上海时区
`docker run --name my-mysql -e MYSQL_ROOT_PASSWORD=123456 -e TZ=Asia/Shanghai -d mysql:8.0`

进入 mysql

```sql
SELECT @@global.time_zone, @@session.time_zone;
SET GLOBAL time_zone = '+08:00';
SET SESSION time_zone = '+08:00';
```

#### 启动 redis 容器

```bash
docker-compose up -d redis # 启动 redis 容器
docker-compose ps # 查看容器状态
docker exec -it <name> redis-cli # 进入容器,用容器的名称<name>这个位置
```

#### 启动停车场管理系统

```bash
docker build -t parkinglot-backend . # 构建镜像
docker run -d -p 8000:8000 --name parkinglot-backend-container parkinglot-backend # 启动容器
docker-compose up --build web # 或者这样构建且启动容器
```

设置映射端口,编辑 docker-compose.yml 文件

```bash
ports:
    - "8080:8080" # 设置映射端口
```

清除重新构建

```bash
docker-compose down -v --remove-orphans
```

#### 关闭容器

关闭所有

```bash
docker-compose down # 关闭容器(要在yml文件所在目录执行)
```

关闭某一个

```bash
docker stop <name>
```

#### 删除容器(删除容器并不会删除镜像)

删除某一个

```bash
docker rm <name>
```

### 创建自定义网络(固定容器 ip,使其内部互通)

```bash
docker network create --subnet=10.1.0.0/24 <name>
```

### 分配 ip 给容器

```bash
docker run --network <name> --ip <ip> --name <name> <image>
docker run -it --network test-net --ip 10.1.0.3 --name c2 91ef0af61f39 /bin/sh # 启动且命令行
```

### 查看网络

```bash
docker network ls
```

#### 查看某个网络的详细信息

```bash
docker network inspect <name>
```

### 删除镜像

删除某一个

```bash
docker rmi <id>
```

### 查看 docker 卷

```bash
docker volume ls # 查看所有卷
```

#### 查看某个卷的详细信息

```bash
docker volume inspect <name>
```

### 删除 docker 卷

删除某一个

```bash
docker volume rm <name>
```

删除不使用的卷

```bash
docker volume prune -f # 删除所有未使用的卷
```

## docker 的理解

1. mysql 镜像: 数据不在容器外部保存,而是保存在容器内部,容器关闭后数据丢失(临时的)
2. 实现数据持久化:
   1. 使用 Docker Volume(卷)
   2. 使用 Docker Bind Mount(绑定挂载)

在 docker-compose.yml 文件中配置

```yml
image: mysql:8.0
environment:
  MYSQL_ROOT_PASSWORD: 123456
  MYSQL_DATABASE: mydb
ports:
  - "3306:3306"
volumes:
  - db_data:/var/lib/mysql # 挂载卷
```

## 奇奇怪怪

Alpine 容器中,安装的软件包默认情况下会被**保存在文件系统中**,因此即使容器重启,curl 仍然可以使用

## docker 里 nginx

主配置文件位置

```bash
/etc/nginx/nginx.conf
# 加载子配置文件(关键)
include /etc/nginx/conf.d/*.conf;
```

站点文件位置(在 conf.d 下配置多个站点)

```bash
/etc/nginx/conf.d/default.conf
/etc/nginx/conf.d/site-a.conf
```

前端打包文件位置

```bash
/usr/share/nginx/html
```

## docker 里 zookeeper(TODO)

## docker 里 kafka(TODO)

## docker 里 redis

```Dockerfile
COPY redis.conf /usr/local/etc/redis/redis.conf
# 启动 Redis 并指定配置文件
CMD ["redis-server", "/usr/local/etc/redis/redis.conf"]
```

## docker 里 mysql

## docker 里 elk

```bash
sysctl vm.max_map_count # 查看最大文件描述符数量 elk至少需要262144
sudo sysctl -w vm.max_map_count=262144 # 设置最大文件描述符数量(重启后失效)

docker run -p 5601:5601 -p 9200:9200 -p 5044:5044 \
  -v /path/to/your/kibana.yml(挂载汉化):/opt/kibana/config/kibana.yml \
  -it --name elk sebp/elk:8.15.1

/opt/kibana/config/kibana.yml # 修改 kibana 配置文件
i18n.locale: "zh-CN" # 设置语言为中文
```

## Dockerfile

1. FROM

```Dockerfile
FROM mysql:8.0 # 指定基础镜像
```

2. LABEL

```Dockerfile
LABEL maintainer="NGINX <nginx@example.com>" # 指定维护者信息
```

3. WORKDIR

```Dockerfile
WORKDIR /app # 指定工作目录(类似cd),如果不存在则创建
```

4. COPY

```Dockerfile
COPY . /app # 将当前目录下的所有文件复制到工作目录下
COPY --from=builder /app/myapp . # 从builder镜像中复制/app/myapp到当前目录下
```

5. RUN(构建镜像时执行)

```Dockerfile
RUN go env -w GOPROXY=https://goproxy.io,direct # 命令行执行命令
```

6. CMD(运行容器时执行)

```Dockerfile
CMD ["go", "run", "main.go"] # 可被
docker run命令覆盖,只能有一个CMD
```

7. EXPOSE

```Dockerfile
EXPOSE 8080 # 声明容器监听的端口,运行时会监听
```

### 在 Dockerfile 同级目录 build 镜像

```bash
docker build -t <name> . # docker build -t my-nginx:1.0(指定镜像名和版本号) .(当前目录)
```

### 清理构建缓存

```bash
docker builder prune
```

## 将镜像推送到 docker hub 完整流程

1. 登录 docker hub

```bash
docker login --username <your-docker-hub-username> --password <your-token>
```

2. docker hub 创建仓库
3. 打标签

```bash
docker tag <image-id> <your-docker-hub-username>/<repository-name>:<tag>
```

示例:

```bash
docker tag parkingmanagementsystem_web:1.0(本地构建出来的镜像名) cailanzz/parkingmanagementsystem-web:1.0(远程仓库的镜像名)
```

4. 推送镜像

```bash
docker push cailanzz/parkingmanagementsystem-web:1.0
```

## 过滤<none>的镜像

```bash
docker images --filter dangling=true -q # -q 只显示镜像ID
```

### 删除<none>的镜像

```bash
docker rmi $(docker images --filter dangling=true -q)
```

## fish 里面 docker 命令补全

![](2025-02-17-15-19-33.png)

## 实践原则

1. 容器只运行一个进程
2. 使用非 root 用户运行进程

```yml
RUN adduser -D appuser # 创建用户
USER appuser # 使用用户
```

3. 不将密钥等敏感信息放在镜像中(环境变量读取)

```bash
docker run -v ./config.yaml:/app/config.yaml
```

4. 多阶段构建

## Contributors |  贡献者

- [CLZZ](https://github.com/zccccc01) 
