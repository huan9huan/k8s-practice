# Kubernetes一键部署利器：kubeadm

## 学习链接：https://time.geekbang.org/column/article/39712

## 学习目的：
> 通过k8s安装的翻墙攻略，理解kube-xxx的安装细节
> 

## step 1: kubeadm的yum安装

### step 1.1: 按照通常的方法，要求加入一个kubernetes.repo的源:

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

安装 kubeadm和kubelet应该成功：
```
yum -y install kubelet kubeadm kubectl 
kubelet --version
```
注意我这里的版本是 v1.12.1，记下这版本号，下面会用。

### step 1.2: 配置k8s.gcr.io的镜像

因为k8s的Static Pod启动需要从 k8s.gcr.io 上拉取必要的镜像，但是这个网站上被封掉了，所以需从别的镜像中拉取这些镜像，然后tag成为 k8s.gcr.io 开头，然后 dockerd就可以从本地拉取了镜像。国内的镜像我使用了 `registry.cn-hangzhou.aliyuncs.com/google_containers` 看起来同步的不错

脚本是：
```
VERSION=v1.12.1
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
kubeadm init --config kubeadm.yaml
```

其中 kubeadm.yaml是
```
apiVersion: kubeadm.k8s.io/v1alpha3
kind: InitConfiguration
kubernetesVersion: "v1.12.1"
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
```

备忘
```
 kubeadm join 106.14.93.74:6443 --token p7jj4u.zdh7j96t6zwqi1z7 --discovery-token-ca-cert-hash sha256:83d3f0240d81664a5033193ff5580eb89da61e5f8e4b263caf99453ff61e1a0a
 ```

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## step 2: 安装网络插件

```
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')&env.IPALLOC_RANGE=20.0.0.0/8"

kubectl get pods -w --all-namespaces
NAMESPACE     NAME                            READY   STATUS             RESTARTS   AGE
kube-system   coredns-576cbf47c7-l9lkw        0/1     Pending            0          9m21s
kube-system   coredns-576cbf47c7-wf87f        0/1     Pending            0          9m21s

===>

kube-system   coredns-576cbf47c7-l9lkw   1/1   Running   0     13m
kube-system   coredns-576cbf47c7-wf87f   1/1   Running   0     13m
```

## step 3: 安装一个worker node节点
