# 学习容器网络的高级知识

## link https://time.geekbang.org/column/article/65287

## 学习目的
> 理解常见的网络开发模式
> UDP的问题所在
> VXLAN的模式的结构
> 通过实践funnel理解网络结构

## debug命令
```
ip neigh show dev weave
bridge fdb show weave
```

## 替换weave为funnal网络
```
rm -rf /etc/cni/net.d/*；

$ kubectl delete -f "https://cloud.weave.works/k8s/net?k8s-version=1.12"
$ vim /etc/kubernetes/manifests/kube-controller-manager.yaml

--allocate-node-cidrs=true
--cluster-cidr=10.244.0.0/16

$ service kubelet restart

kubectl create -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml

kubectl delete -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml

kubectl get ds --all-namespaces
kubectl get pods --all-namespaces -o wide


kubectl get nodes -o wide
```

## 确认新的route产生了
```
$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         gateway         0.0.0.0         UG    0      0        0 eth1
10.0.0.0        10.28.99.247    255.0.0.0       UG    0      0        0 eth0
10.28.96.0      0.0.0.0         255.255.252.0   U     0      0        0 eth0
10.244.0.0      0.0.0.0         255.255.255.0   U     0      0        0 cni0

$ ip route
default via 139.196.147.247 dev eth1 

10.244.0.0/24 dev cni0 proto kernel scope link src 10.244.0.1 
```

## 观察docker0和funnel0的关系

ip link delete docker0
ip link delete cni0
route

service docker stop
service docker start

journalctl -u docker -f

```
$ sudo mkdir -p /var/run/flannel/
$ sudo cat <<EOF > /var/run/flannel/subnet.env 
FLANNEL_NETWORK=192.168.0.0/16
FLANNEL_SUBNET=192.168.0.1/24
FLANNEL_MTU=1450
FLANNEL_IPMASQ=true
EOF
```

kubeadm join 139.196.145.43:6443 --token 27tuvw.hnt6noyrtcob6hys --discovery-token-unsafe-skip-ca-verification

ifconfig发现flannel.1
flannel.1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1450
        inet 10.244.0.0  netmask 255.255.255.255  broadcast 0.0.0.0
        ether 56:73:c0:a3:4c:24  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ip link del cni0
ip link del flannel.1
rm -fr /var/run/flannel/subnet.env 

发现flannel运行在其他的node上
 kubectl get pods --all-namespaces -o wide

sudo kubeadm reset -f && sudo rm -fr /etc/kubernetes/

注意：如下文件被拷贝到pi中，但是目的并不明确
/etc/cni/net.d/10-flannel.conflist 