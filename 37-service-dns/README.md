## Service/DNS/服务发现

## link: https://time.geekbang.org/column/article/68636

## 学习目的
1. service的内部机制是什么
2. ipvs是怎么回事


## env
```
machine: d2 & v1

# on d2:
kubeadm init --pod-network-cidr=10.244.0.0/16

kubectl create -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml


ifconfig |grep  -A 5 flannel.1

kubectl get nodes
## should be ready

# on v1, add v1 into cluster
kubeadm join 138.68.29.180:6443 --token mzfj4j.k8mm7v0cr3jm2tfe --discovery-token-ca-cert-hash sha256:45ca2a8707a8382c61efa572fb343218cf767d960899e62e81e8bb263b66c618

# wait for v1 be ready
kubectl get nodes -w
v1    Ready   <none>   10s   v1.12.2
```

### task 1: 使用hostname的deployment理解service的实现
```
kubectl create -f ./hostname.yaml
kubectl get pods -o wide
## 应该得到三个pod，并且是running的状态

# 查找service的entrypoints

```