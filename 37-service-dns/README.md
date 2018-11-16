## Service/DNS/服务发现

## link: https://time.geekbang.org/column/article/68636

## 学习目的
1. service的内部机制是什么，熟悉svc的endpoint，vip等，以及和iptables之间的关系
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
$ kubectl create -f ./hostname.yaml
$ kubectl get pods -o wide
## 应该得到三个pod，并且是running的状态

# 查找service的entrypoints
$ kubectl get endpoints hostnames
NAME        ENDPOINTS                                         AGE
hostnames   10.244.1.5:9376,10.244.1.6:9376,10.244.1.7:9376   33s

# 访问不同的pod得到不同的hostname
$ kubectl get svc hostnames
NAME        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
hostnames   ClusterIP   10.103.184.52   <none>        80/TCP    2m18s

# 访问这个service的地址VIP
$ curl 10.103.184.52:80


# 从iptables-save看到这个rule
sudo iptables-save |grep hostname
-A KUBE-SERVICES ! -s 10.244.0.0/16 -d 10.103.184.52/32 -p tcp -m comment --comment "default/hostnames:default cluster IP" -m tcp --dport 80 -j KUBE-MARK-MASQ
-A KUBE-SERVICES -d 10.103.184.52/32 -p tcp -m comment --comment "default/hostnames:default cluster IP" -m tcp --dport 80 -j KUBE-SVC-ODX2UBAZM7RQWOIU
## 意思是说，凡是dest的ip是10.103.184.52/32端口dport是80的，都jump到KUBE-SVC-ODX2UBAZM7RQWOIU的规则中

# 继续观察被jump到的chain的规则
$ sudo iptables-save |grep KUBE-SVC-ODX2UBAZM7RQWOIU
-A KUBE-SVC-ODX2UBAZM7RQWOIU -m statistic --mode random --probability 0.33332999982 -j KUBE-SEP-LZSOAAIF344LOQYR
-A KUBE-SVC-ODX2UBAZM7RQWOIU -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-TKZFUYKNSRMALYNM
-A KUBE-SVC-ODX2UBAZM7RQWOIU -j KUBE-SEP-7MW3KEEC4RCNSIJ2
## 注意这里的三个概率值的分配

$ sudo iptables-save |grep KUBE-SEP-7MW3KEEC4RCNSIJ2
-A KUBE-SEP-7MW3KEEC4RCNSIJ2 -s 10.244.1.7/32 -j KUBE-MARK-MASQ
-A KUBE-SEP-7MW3KEEC4RCNSIJ2 -p tcp -m tcp -j DNAT --to-destination 10.244.1.7:9376

```

## task2：ipvs提升性能
TODO