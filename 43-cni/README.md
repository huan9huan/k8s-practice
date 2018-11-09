# 理解cni和network plugin

# link https://time.geekbang.org/column/article/67351


## 学习目的 - 理解插件

## task 1.0: 新建立一个可以工作的集群，理解网络结构

```
$ kubeadm init --pod-network-cidr=10.244.0.0/16

# verify: control-manager conf中有cidr
$ grep cidr /etc/kubernetes/manifests/kube-controller-manager.yaml
    - --allocate-node-cidrs=true
    - --cluster-cidr=10.244.0.0/16
    - --node-cidr-mask-size=24
## 此时，cni0还没生成
$ route |grep cni0
<空>

$ kubectl create -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml

## cni0生成
route |grep cni0
10.244.0.0      0.0.0.0         255.255.255.0   U     0      0        0 cni0

## flannel.1 也生成
$ ifconfig |grep  -A 5 flannel.1
flannel.1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
        inet 10.244.0.0  netmask 255.255.255.255  broadcast 0.0.0.0
        ether 52:ac:9d:83:55:dd  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)

```

## task 1.1 - worker node中使用flannel的网络
```
kubeadm join 139.196.145.43:6443 --token jsaerg.p466dllqxaidkow7 --discovery-token-ca-cert-hash sha256:0ab2893e5dc413e7a1b4578421e2b7b1b5d851115c89679116c7a4d7c81871d1

n1 $ route -n |grep cni0
10.244.2.0      0.0.0.0         255.255.255.0   U     0      0        0 cni0

n1 $ route -n |grep flannel.1
10.244.0.0      10.244.0.0      255.255.255.0   UG    0      0        0 flannel.1

master $ kubectl get nodes

## 在node上创建一个pod
master $ cd /root/k8s/14-pod-basic
master $ kubectl create -f ./simple-pod.yaml 
master $ kubectl describe pod simple-pod
Node:               mysql/10.29.25.61
IP:                 10.244.2.2
## 表明基于flannel的网络生成

```

## task 3: 跨网络实现pod互访问，加入cvhex节点
按照 文档 10 中的步凑安装k8s
```
kubeadm join 139.196.145.43:6443 --token jsaerg.p466dllqxaidkow7 --discovery-token-ca-cert-hash sha256:0ab2893e5dc413e7a1b4578421e2b7b1b5d851115c89679116c7a4d7c81871d1

## 等待cvhex的node是ready的状态
kubectl get pods --all-namespaces


# run一个node上的nginx：
$ kubectl create -f ./simple-pod.yaml 


$ kubectl get pods -o wide
NAME         READY   STATUS    RESTARTS   AGE   IP           NODE    NOMINATED NODE
simple-pod   2/2     Running   0          16m   10.244.4.2   cvhex   <none>

# 注意到ip和node

cvhex $ route |grep flannel.1
10.244.0.0      10.244.0.0      255.255.255.0   UG    0      0        0 flannel.1
cvhex $ route |grep cni0
10.244.4.0      *               255.255.255.0   U     0      0        0 cni0

```
其中的一些坑：
> 1. 要注意查看kubelet的错误 `journalctl -u kubelet -f`   
> 2. kube-proxy的镜像需要在本地准备好，特别是版本问题，建议把多个版本都sync下来  
> 3. `/var/run/flannel/subnet.env` 会自己被准备好  
> 4. 第一次pod会很久，flannel的镜像拉取可能需要点时间，然后才能生成cni0

## 附录： 一些debug的命令帮助troubleshots
```
kubectl get pods --all-namespaces
kubectl describe pod  kube-flannel-ds-amd64-c4h7w -n kube-system
kubectl logs  kube-flannel-ds-amd64-c4h7w -n kube-system
kubectl logs  kube-flannel-ds-arm-m6bxh  -n kube-system

journalctl -u kubelet -f

# 查看subnet的位置
$ cat /var/run/flannel/subnet.env 
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.0.1/24
FLANNEL_MTU=1450
FLANNEL_IPMASQ=true

# 查看cni插件信息
cat /etc/cni/net.d/10-flannel.conflist 

```

### 彻底清除
```
kubeadm reset -f && rm -fr $HOME/.kube && rm -fr /etc/kubernetes/

rm -fr /var/run/flannel && ip link del cni0 && ip link del flannel.1
```