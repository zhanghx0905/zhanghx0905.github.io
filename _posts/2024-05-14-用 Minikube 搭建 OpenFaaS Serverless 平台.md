---
title: Minikube 搭建 OpenFaaS Serverless 平台
tags: Minikube OpenFaaS Serverless
---


最近对云函数很感兴趣，研究了两天，用 Minikube 搭建了 GitHub 上 star 最多的 Serverless 框架 OpenFaaS。

在这一番折腾后，我发现该框架并不能满足我的需求。我需要的不是“弹性伸缩”降低服务器成本，而是加快服务上线流程，免去运维的技术负债，提高团队的生产力。

在此记录一下 OpenFaaS 的安装过程。

安装前，需确保服务器已经安装 Docker。

```sh
# Install Docker socat conntrack
sudo apt-get update 
sudo apt-get install docker.io socat conntrack -y
```

将 minikube kubectl arkade faas-cli 四个文件从互联网下载到本地。

```sh
sudo install minikube-linux-amd64 /usr/local/bin/minikube
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

chmod +x arkade
sudo cp arkade /usr/local/bin/

chmod +x faas-cli
sudo cp faas-cli /usr/local/bin/
sudo ln -s /usr/local/bin/faas-cli /usr/local/bin/faas
```

在非 root 账户下,启动 minikube

```sh
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kicbase:v0.0.43

minikube start --image-mirror-country=cn --kubernetes-version=1.23.0 --base-image=registry.cn-hangzhou.aliyuncs.com/google_containers/kicbase:v0.0.43 --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers 
```

安装 openfaas，等待镜像拉取完成。如果提示 sudo 找不到 arkade，可以切到 root 用户执行。

```sh
sudo arkade install openfaas --kubeconfig /home/$USERNAME/.kube/config
```

安装完成后，按照 arkade 的提示执行如下配置：

```sh
# Forward the gateway to your machine
kubectl rollout status -n openfaas deploy/gateway
kubectl port-forward -n openfaas svc/gateway 8080:8080 &

PASSWORD=$(kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode; echo)
echo -n $PASSWORD | faas-cli login --username admin --password-stdin
```

此时打开 localhost:8080，能够登录到 OpenFaaS 管理页。