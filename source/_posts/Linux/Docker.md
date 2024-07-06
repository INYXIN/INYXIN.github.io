---
title: Docker
date: 2024-06-01 ‏‎23:41:19
tags: [ Docker ]
categories: [ Linux ]

---

## 镜像

- `docker version` 显示docekr的详细信息
- `docker info` 显示docker的系统信息
- `docker --help` docker的命令帮助手册

- `docker search {关键字}` 搜索镜像

  ```sh
  docker search nginx #搜索nginx镜像
  docker search openjdk #搜索openjdk镜像
  ```

- `docker pull {镜像名[标签]}` 拉取镜像

  ```sh
  docker pull nginx #拉取 nginx镜像
  docker pull openjdk:latest #拉取 openjdk镜像
  ```

- `docker images` 查看已经下载的镜像

- `docker rmi {镜像名或镜像ID}` 删除镜像

  ```sh
  docker rmi nginx # 删除镜像名为nginx的镜像
  docker rmi e235nd # 根据镜像ID删除
  ```

- `docker tag {原来的镜像名[:标签]} {新的镜像名[:标签]}` 复制镜像并修改名称

  ```sh
  docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
  #SOURCE_IMAGE 是要标记的现有镜像的名称或 ID。
  #TARGET_IMAGE 是为现有镜像指定的新标签的名称。
  #TAG 是可选的标签，用于指定版本或其他特定标识符。
  例如:
  docker tag nginx inyxin/nginx:beta  
  ```

- `docker load -i {镜像的tar}`  导入镜像

  ```sh
  docker load -i /root/docker-centos-httpd.tar
  ```

- `docker save -o {镜像的tar} {镜像名}`  导出镜像

  ```sh
  docker save -o xxx.tar 镜像名
  docker save -o my_image.tar my_image 
  ```

## 容器

- `docker run [参数] {镜像} [COMMAND] [ARG...]` **启动容器 (重点)**

  ```sh
  #语法
  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
  #[OPTIONS]
  -d, --detach：在后台运行容器。
  --name：为容器指定一个名称。
  -p, --publish：将容器端口映射到宿主机端口。
  -v, --volume：挂载宿主机目录到容器内部。
  -e, --env：设置环境变量。
  --network：连接容器到指定的网络。
  --restart：在容器退出时指定重启策略。
  --rm：容器退出时自动删除。
  -it：交互式操作，通常与 -i（标准输入）和 -t（终端）一起使用。
  # [COMMAND] [ARG...]
  #可以覆盖默认的命令和参数。例如，你可以执行以下命令来覆盖默认的命令：
  docker run my_image echo "Hello, Docker!"
  #在这个例子中，echo "Hello, Docker!" 将覆盖镜像中定义的默认命令，容器将输出 Hello, Docker! 而不是 Hello, World!。
  ```

- `docker build [OPTIONS]` 创建镜像

  ```sh
  docker build [OPTIONS]
  #[OPTIONS]
  -f : 指定dockerfile文件路径 , 默认就是当前目录的 Dockerfile 文件
  -t : 构建后的镜像名以及标签
  #[EXAMPLE]
  docker build -t inyxin/openjdk:1.8 
  ```

- `docker logs {容器ID}`  查看容器日志

- `docker rename {旧名字} {新名字}` 容器重命名

- `docker ps [-a]`  查看正在运行的容器 [ -a 查看全部容器 ] 

- `docekr kill {容器名 或 容器ID}` 杀死一个容器

- `docker rm {容器名 或 容器ID}` 删除容器 

[^{容器名 或 容器ID}]: 以下简称 {容器ID}

- `docker history {容器ID}`  查看docker 镜像的变更历史

- `docker start {容器ID}` 启动一个容器

- `docker restart {容器ID}`  重启容器

- `docker stop` 停止容器

- `doceker image inspect {容器ID}`  查看容器内源数据

- `docker cp {容器ID}:路径 主机路径` 从容器内拷贝文件到主机

- `docker exec -it {容器名ID} /bin/bash` 进入容器

  ```
  docker exec -it 1Panel-minio-UuPy /bin/bash
  ```

- `docker commit -m="提交的描述信息"  -a="作者"  {容器id}  {目标镜像名:[TAG]}`

```sh
docker run \
--name inyxin-minio \ 
-v "/opt/1panel/apps/minio/minio/data:/data" \ 
-v "/opt/1panel/apps/minio/minio/certs:/root/.minio/certs" \
-p "9001:9001" \ 
-p "9000:9000" \
--restart always \ 
-e MINIO_ROOT_PASSWORD="1423716216@qq.com" \ 
-e MINIO_ROOT_USER="1423716216@qq.com" \
-e MINIO_SERVER_URL="http://127.0.0.1:9000" \
-e MINIO_BROWSER=on \
-e MINIO_BROWSER_LOGIN_ANIMATION=on \
-e MINIO_BROWSER_REDIRECT_URL="http://127.0.0.1:9001" \
-e MINIO_BROWSER_SESSION_DURATION=12h \
-e PATH=/opt/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin \
-e container=oci \
-e MINIO_ACCESS_KEY_FILE=access_key \
-e MINIO_SECRET_KEY_FILE=secret_key \
-e MINIO_ROOT_USER_FILE=access_key \
-e MINIO_ROOT_PASSWORD_FILE=secret_key \
-e MINIO_KMS_SECRET_KEY_FILE=kms_master_key \
-e MINIO_UPDATE_MINISIGN_PUBKEY=RWTx5Zr1tiHQLwG9keckT0c45M3AGeHD6IvimQHpyRywVWGbP1aVSGav \
-e MINIO_CONFIG_ENV_FILE=config.env \
minio/minio 

```

## Dockerfile

```sh
# 指定基础镜像，例如 node:14。这是 Dockerfile 的第一条指令，定义了新镜像是基于哪个已有镜像创建的。
FROM <base_image>

# 设置作者信息 (可选)
LABEL maintainer="<your_name> <your_email>"

# 设置工作目录。所有后续的 COPY, ADD, RUN 等指令都将在这个目录下执行。例如 WORKDIR /usr/src/app
WORKDIR /usr/src/app

# 复制文件到镜像中
COPY <source> <destination>

# 下载并安装依赖包
RUN <command>

# 设置环境变量 (可选)
ENV <key>=<value>

# 暴露端口 (可选)
EXPOSE <port>

# 运行容器启动时的命令 ,与 CMD 类似，但 ENTRYPOINT 不会被 docker run 提供的参数覆盖。例如：
ENTRYPOINT ["<command>"]

# 指定默认参数 (可选) 指定容器启动时要执行的命令。例如，CMD ["npm", "start"]。注意，CMD 只会有一个生效，如果存在多个 CMD 指令，只有最后一个会被执行。
CMD ["<args>"]
```

通过 `docker build -t {镜像:标签} 即可构建

## 

## 存储卷

```yaml
Usage:  docker volume COMMAND

Manage volumes

Commands:
  create      Create a volume
  inspect     Display detailed information on one or more volumes
  ls          List volumes
  prune       Remove unused local volumes
  rm          Remove one or more volumes
```

## 网络

```yaml
Usage:  docker network COMMAND

Manage networks

Commands:
  connect     Connect a container to a network
  create      Create a network
  disconnect  Disconnect a container from a network
  inspect     Display detailed information on one or more networks
  ls          List networks
  prune       Remove all unused networks
  rm          Remove one or more networks
```

## DockerCompose

```yaml
Usage:  docker compose [OPTIONS] COMMAND

Define and run multi-container applications with Docker

Options:
      --all-resources              Include all resources, even those not used by services
      --ansi string                Control when to print ANSI control characters ("never"|"always"|"auto") (default "auto")
      --compatibility              Run compose in backward compatibility mode
      --dry-run                    Execute command in dry run mode
      --env-file stringArray       Specify an alternate environment file
  -f, --file stringArray           Compose configuration files
      --parallel int               Control max parallelism, -1 for unlimited (default -1)
      --profile stringArray        Specify a profile to enable
      --progress string            Set type of progress output (auto, tty, plain, quiet) (default "auto")
      --project-directory string   Specify an alternate working directory
                                   (default: the path of the, first specified, Compose file)
  -p, --project-name string        Project name

Commands:
  attach      Attach local standard input, output, and error streams to a service's running container
  build       Build or rebuild services
  config      Parse, resolve and render compose file in canonical format
  cp          Copy files/folders between a service container and the local filesystem
  create      Creates containers for a service
  down        Stop and remove containers, networks
  events      Receive real time events from containers
  exec        Execute a command in a running container
  images      List images used by the created containers
  kill        Force stop service containers
  logs        View output from containers
  ls          List running compose projects
  pause       Pause services
  port        Print the public port for a port binding
  ps          List containers
  pull        Pull service images
  push        Push service images
  restart     Restart service containers
  rm          Removes stopped service containers
  run         Run a one-off command on a service
  scale       Scale services
  start       Start services
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop services
  top         Display the running processes
  unpause     Unpause services
  up          Create and start containers
  version     Show the Docker Compose version information
  wait        Block until the first service container stops
  watch       Watch build context for service and rebuild/refresh containers when files are updated
```

