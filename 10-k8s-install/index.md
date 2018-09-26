# k8s安装的翻墙攻略

学到了k8s的安装，就碰到了棘手的翻墙问题（我是在阿里云上CentOS实验和学习的），核心问题是两个：
> 1. kubeadm的安装，因为yum/apt源使用packages.cloud.google.com而无法访问的问题
> 2. gcr.io 无法访问从而造成k8s启动所必须的的images无法拉取
> 3. kubeadm init的时候，远程探测版本撞墙问题

一个个解决。

## Problem 1: kubeadm的yum安装
方法尝试用了两个，一个是shadowsocks代理（前提你要有国外的vps），一个是使用aliyun自己的yum镜像，后者相对简单一些。

按照通常的方法，要求加入一个kubernetes.repo的源:

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
#baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
EOF

yum -y install epel-release
yum clean all
yum makecache
```

然后安装 kubeadm和kubelet应该成功：
```
yum -y install kubelet kubeadm kubectl 
kubelet --version
```
注意我这里的版本是 v1.11.3，下面会用。

## Problem 2: k8s.gcr.io 访问问题
因为k8s的Static Pod启动需要从 k8s.gcr.io 上拉取必要的镜像，但是这个网站上被封掉了，所以需从别的镜像中拉取这些镜像，然后tag成为 k8s.gcr.io 开头，然后 dockerd就可以从本地拉取了镜像。国内的镜像我使用了 `registry.cn-hangzhou.aliyuncs.com/google_containers` 看起来同步的不错。

运行脚本是这样的：
```
MY_REGISTRY=registry.cn-hangzhou.aliyuncs.com/google_containers
#registry.cn-hangzhou.aliyuncs.com/google-images
VERSION=v1.11.3

## 拉取镜像
docker pull ${MY_REGISTRY}/kube-apiserver-amd64:${VERSION}
docker pull ${MY_REGISTRY}/kube-controller-manager-amd64:${VERSION}
docker pull ${MY_REGISTRY}/kube-scheduler-amd64:${VERSION}
docker pull ${MY_REGISTRY}/kube-proxy-amd64:${VERSION}
docker pull ${MY_REGISTRY}/etcd-amd64:3.2.18
docker pull ${MY_REGISTRY}/pause-amd64:3.1
docker pull ${MY_REGISTRY}/coredns:1.1.3
docker pull ${MY_REGISTRY}/pause:3.1

## 添加Tag
docker tag ${MY_REGISTRY}/kube-apiserver-amd64:${VERSION} k8s.gcr.io/kube-apiserver-amd64:${VERSION}
docker tag ${MY_REGISTRY}/kube-scheduler-amd64:${VERSION} k8s.gcr.io/kube-scheduler-amd64:${VERSION}
docker tag ${MY_REGISTRY}/kube-controller-manager-amd64:${VERSION} k8s.gcr.io/kube-controller-manager-amd64:${VERSION}
docker tag ${MY_REGISTRY}/kube-proxy-amd64:${VERSION} k8s.gcr.io/kube-proxy-amd64:${VERSION}
docker tag ${MY_REGISTRY}/etcd-amd64:3.2.18 k8s.gcr.io/etcd-amd64:3.2.18
docker tag ${MY_REGISTRY}/pause-amd64:3.1 k8s.gcr.io/pause-amd64:3.1
docker tag ${MY_REGISTRY}/coredns:1.1.3 k8s.gcr.io/coredns:1.1.3
docker tag ${MY_REGISTRY}/pause:3.1 k8s.gcr.io/pause:3.1
```

注意：
> 1. 不同的版本需要特定version的image，如果长期跟踪kubeadm和kubectl，要注意维护这个image列表  
> 2. 如果使用代理方案，注意 `http_proxy=<proxy address>:<proxy port> docker pull` 并不能生效，而是要让docker daemon感知到proxy的存在。这是一个坑点，但不是docker的设计缺陷，而是image pull的操作是docker服务进程管理的，当然代理要让这个进程使用。

### Problem 3: 一个小尾巴，关闭版本探测
```
kubeadm init --kubernetes-version=v1.11.3
```
否则kubeadm会访问一个墙外的文件，找这个版本， 也会卡住。

然后就可以愉快的玩k8s了，真呀嘛真好用，不浪费这一番折腾。

### 墙很害人，但是墙让人更加强壮，不会翻墙，就被淘汰