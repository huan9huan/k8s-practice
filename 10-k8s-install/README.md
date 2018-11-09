# Kubernetes一键部署利器：kubeadm

## 学习链接：https://time.geekbang.org/column/article/39712

## 学习目的：
> 通过k8s安装的翻墙攻略，理解kube-xxx的安装细节

## step 1: kubeadm的yum安装

### step 1.1: 按照通常的方法，要求加入一个kubernetes.repo的源:

centos:
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

ubuntu:
```
sudo apt-get update
sudo apt-get install apt-transport-https ca-certificates -y

cat /etc/apt/sources.list
# 系统安装源
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted
deb http://mirrors.aliyun.com/ubuntu/ xenial universe
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates universe
deb http://mirrors.aliyun.com/ubuntu/ xenial multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
# kubeadm及kubernetes组件安装源
deb https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial main

cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb http://mirrors.ustc.edu.cn/kubernetes/apt kubernetes-xenial main
EOF

sudo apt-get update
apt-get install docker.io
apt-get install -y kubelet kubeadm kubectl --allow-unauthenticated

```

安装 kubeadm和kubelet应该成功：
```
# centos
yum -y install kubelet kubeadm kubectl 

# in ubuntu
# apt-get install -y kubelet kubeadm kubectl 
kubelet --version
```
注意我这里的版本是 v1.12.2，记下这版本号，下面会用。

### step 1.2: 配置k8s.gcr.io的镜像

因为k8s的Static Pod启动需要从 k8s.gcr.io 上拉取必要的镜像，但是这个网站上被封掉了，所以需从别的镜像中拉取这些镜像，然后tag成为 k8s.gcr.io 开头，然后 dockerd就可以从本地拉取了镜像。国内的镜像我使用了 `registry.cn-hangzhou.aliyuncs.com/google_containers` 看起来同步的不错

脚本是：
```
VERSION=v1.12.2
ETCD_VERSION=3.2.24
COREDNS_VERSION=1.2.2
PAUSE_VERSION=3.1
MY_REGISTRY=registry.cn-hangzhou.aliyuncs.com/google_containers

## 拉取镜像
docker pull ${MY_REGISTRY}/kube-apiserver:${VERSION}
docker pull ${MY_REGISTRY}/kube-controller-manager:${VERSION}
docker pull ${MY_REGISTRY}/kube-scheduler:${VERSION}
docker pull ${MY_REGISTRY}/kube-proxy:${VERSION}
docker pull ${MY_REGISTRY}/etcd:${ETCD_VERSION}
docker pull ${MY_REGISTRY}/coredns:${COREDNS_VERSION}
docker pull ${MY_REGISTRY}/pause:${PAUSE_VERSION}

## 添加Tag
docker tag ${MY_REGISTRY}/kube-apiserver:${VERSION} k8s.gcr.io/kube-apiserver:${VERSION}
docker tag ${MY_REGISTRY}/kube-scheduler:${VERSION} k8s.gcr.io/kube-scheduler:${VERSION}
docker tag ${MY_REGISTRY}/kube-controller-manager:${VERSION} k8s.gcr.io/kube-controller-manager:${VERSION}
docker tag ${MY_REGISTRY}/kube-proxy:${VERSION} k8s.gcr.io/kube-proxy:${VERSION}
docker tag ${MY_REGISTRY}/etcd:${ETCD_VERSION} k8s.gcr.io/etcd:${ETCD_VERSION}
docker tag ${MY_REGISTRY}/coredns:${COREDNS_VERSION} k8s.gcr.io/coredns:${COREDNS_VERSION}

docker tag ${MY_REGISTRY}/pause:${PAUSE_VERSION} k8s.gcr.io/pause:${PAUSE_VERSION}
```

### step 1.3： 使用kubeadm初始化kubernetes
```
$ kubeadm init

sudo kubeadm join 139.196.145.43:6443 --token af53r6.co1z7sa33iltultn --discovery-token-ca-cert-hash sha256:24c043a2fb91cf995166e075562064b1608aedc725d132b8a5e5294b24eec213
```

并根据kubeadm的数据，配置config
```
rm -fr ~/.kube
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

问题记录：   
> 如果没有配置config，可能无法运行kubectl  
> 确保自定义podCidr
```
$ kubectl cluster-info dump | grep -i cidr
```

## step 2: 安装网络插件

```
$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')&env.IPALLOC_RANGE=20.0.0.0/16"

$ kubectl logs -f weave-net-8hr4s weave -n kube-system
$ kubectl exec -n kube-system weave-net-9zv45 -c weave -- /home/weave/weave --local status

$ kubectl get pods -w --all-namespaces
NAMESPACE     NAME                            READY   STATUS             RESTARTS   AGE
kube-system   coredns-576cbf47c7-l9lkw        0/1     Pending            0          9m21s
kube-system   coredns-576cbf47c7-wf87f        0/1     Pending            0          9m21s

===>

kube-system   coredns-576cbf47c7-l9lkw   1/1   Running   0     13m
kube-system   coredns-576cbf47c7-wf87f   1/1   Running   0     13m


$ kubectl describe pods -n kube-system
```

## step 3: 安装一个worker node节点

### Step 3.1 安装kubeadm，参照 Step 1.1

### Step 3.2 加入节点
```
$ kubeadm join xxx
service kubelet status -l
```

坑点1：打开端口6443，否则会造成nodelost

### Step 3.3 检查
```
# in master
$ kubectl get nodes -w

# in node:
$ systemctl restart kubelet
$ service kubelet status -l
```

问题记录
> master的ip不可以随便
> 要注意检查端口是否可通
> 可能cni无法初始化，fix的方法：
```
$ cat /etc/cni/net.d/10-weave.conf
{
    "name": "weave",
    "type": "weave-net",
    "hairpinMode": true
}
```

## Step 4: 让master也是一个节点
```
$ kubectl describe node mysql
...
Taints:             node-role.kubernetes.io/master:NoSchedule
...

$ kubectl taint nodes --all node-role.kubernetes.io/master-
node/mysql untainted 

# 确认
$ kubectl describe node mysql
Taints： <None>

```

## Step 5: 安装Rook从而支持Ceph

```
$ kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/operator.yaml

$ kubectl apply -f https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/ceph/cluster.yaml

$ kubectl get pods --all-namespaces
$ kubectl get pods -n rook-ceph-system
$ kubectl get pods -n rook-ceph
```

## 清理重置系统的命令
慎用！！
```
$ kubeadm reset -f
$ rm -fr ~/.kube
```

## 坑
### 关闭掉firewalld
```
service firewalld status
```