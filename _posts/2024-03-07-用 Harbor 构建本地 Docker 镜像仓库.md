---
title: 用 Harbor 构建本地 Docker 镜像仓库
tags: Harbor Docker Cloud
---


在团队的 Docker 镜像仓库失效后，我决定使用 Harbor 在本地服务器上重新搭建一个镜像仓库，便于管理和分发 Docker 镜像。以下是从安装到使用的整个过程。


## 安装 Harbor

### 安装前提

安装 Harbor 前，需确保服务器已经安装 Docker 和 Docker Compose。

我使用的两台服务器均满足此要求，因此在此略去。

### 1. 下载 Harbor 安装包

从 [Harbor 的 Github Release 页面](https://goharbor.io/docs/2.10.0/)下载 Offline 安装包（版本 v2.10.0）。

```sh
tar xzvf harbor-offline-installer-v2.10.0.tgz
cd harbor
```

### 2. 配置 Harbor

编辑 `harbor.yml` 文件以配置 Harbor：

```sh
vim harbor.yml
```

重要配置项：

- `hostname`: 服务器 IP；
- `http - port`: Harbor 使用的 HTTP 端口；
- `https`: 此次配置不使用 SSL，因此将相关配置注释掉；
- `harbor_admin_password`: 管理员账户密码。

### 3. 安装 Harbor

执行 `install.sh` 脚本，利用 Docker Compose 实现一键安装。

```sh
./install.sh
```

安装完成后，通过浏览器访问 Harbor 端口，即可看到登录界面。

## 使用 Harbor

### 配置 Docker

修改 `daemon.json`，为 Docker 添加不安全的仓库地址：

```json
{
  "insecure-registries": [
    "ip:port"
  ]
}
```

Linux 系统下，`daemon.json` 位于 `/etc/docker/` 目录。

```sh
sudo vim /etc/docker/daemon.json
```

Windows 下，在 Docker Desktop 配置项中修改。

修改后重启 Docker 守护进程和服务，在 Linux 中执行如下命令：

```sh
sudo systemctl restart docker
```

在 Windows 中重启 Docker Desktop。

### 登录 Harbor

使用以下命令登录到 Harbor：

```sh
docker login ip:port -u <username> -p <password>
```

### 推送和拉取镜像

- 给镜像打标签，添加 Harbor 前缀：

  ```sh
  docker tag redis:7.2.3 ip:port/library/redis:7.2.3
  ```

- 推送镜像到 Harbor：

  ```sh
  docker push ip:port/library/redis:7.2.3
  ```

- 从 Harbor 拉取镜像：

  ```sh
  docker pull ip:port/library/redis:7.2.3
  ```

### 拉取来自其它源的镜像

在 Harbor 中，登录管理员账户后，可以增加新的镜像源，并在项目管理中创建同名项目。

配置镜像源后，初次拉取新镜像时， Harbor 会从远端仓库获取镜像。后续再拉取时，能直接使用 Harbor 缓存的本地镜像。

- 对于 DockerHub 的镜像，使用私服地址拉取：

  ```sh
  docker pull ip:port/dockerhub/nginx
  ```

- 对于 GitHub Container Registry 的镜像，同样通过私服地址拉取：

  ```sh
  docker pull ip:port/ghcr.io/username/image:tag
  ```

- 对于 NVIDIA Container Registry 的镜像，通过私服地址拉取：

  ```sh
  docker pull ip:port/nvcr.io/nvidia/pytorch:24.01-py3
  ```

