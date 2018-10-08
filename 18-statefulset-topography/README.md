## 学习stateful的拓扑结构

### link https://time.geekbang.org/column/article/41017

## 学习目的：
> 理解headless service的DNS的构造, 并验证如何在stateful set中创建这种service
> 理解statefulset如何使用headless保持pod的拓扑结构

# 实验1 ： headless service的dns构造的实践

```
## 创建一个headless service
$ kubectl create -f svc.yaml
$ kubectl get service nginx
NAME      TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
nginx     ClusterIP   None         <none>        80/TCP    50s

## 创建一个statefulset
$ kubectl create -f ss-web.yaml
$ kubectl get statefulset ss-web

NAME      DESIRED   CURRENT   AGE
ss-web    2         2         3m

## 立即观察创建状态
$ kubectl get pods -w -l app=nginx

NAME       READY     STATUS    RESTARTS   AGE
ss-web-0   0/1       Pending   0         0s
ss-web-0   0/1       Pending   0         0s
ss-web-0   0/1       ContainerCreating   0         0s
ss-web-0   1/1       Running   0         2s
ss-web-1   0/1       Pending   0         0s
ss-web-1   0/1       Pending   0         0s
ss-web-1   0/1       ContainerCreating   0         0s
ss-web-1   1/1       Running   0         2s

## 进入到pod的network中观察hostname
$ kubectl exec ss-web-0 -- sh -c 'hostname'
web-0
$ kubectl exec ss-web-1 -- sh -c 'hostname'
web-1

kubectl exec ss-web-0 -it -- sh

ping ss-web-0.nginx

## 进一步的创建一个临时性的pod，然后执行nslookup观察dns的情况
$ kubectl run -i --tty --image busybox dns-test --restart=Never --rm /bin/sh 

## 如下的nslookup是失败的
/ # nslookup ss-web-0.nginx
Server:		10.96.0.10
Address:	10.96.0.10:53
** server can't find ss-web-0.nginx: NXDOMAIN

## 如下的ping是成功的
/ # ping ss-web-1.nginx
PING ss-web-1.nginx (20.0.0.12): 56 data bytes
64 bytes from 20.0.0.12: seq=0 ttl=64 time=0.066 ms
...

## 或者全名，也是可以的
/ # ping ss-web-1.nginx.default.svc.cluster.local

```

** todo: 需要找到为什么会失败的原因 **

# 实验2：通过删除pod理解k8s如何保持状态的

> 实验1中，ss-web-0的ip是20.0.0.10，ss-web-1的ip是20.0.0.12

```
# 首先启动一个console，持续观察pod的运行情况
kubectl get pod -w -l app=nginx

NAME       READY     STATUS    RESTARTS   AGE
ss-web-0   1/1       Running   0          4h
ss-web-1   1/1       Running   0          4h

---- 分割线：删除后的相应 -----
ss-web-0   1/1       Terminating   0         4h
ss-web-1   1/1       Terminating   0         4h
ss-web-1   0/1       Terminating   0         4h
ss-web-0   0/1       Terminating   0         4h
ss-web-1   0/1       Terminating   0         4h
ss-web-1   0/1       Terminating   0         4h
ss-web-0   0/1       Terminating   0         4h
ss-web-0   0/1       Terminating   0         4h
ss-web-0   0/1       Pending   0         0s
ss-web-0   0/1       Pending   0         0s
ss-web-0   0/1       ContainerCreating   0         0s
ss-web-0   1/1       Running   0         0s
ss-web-1   0/1       Pending   0         0s
ss-web-1   0/1       Pending   0         0s
ss-web-1   0/1       ContainerCreating   0         0s
ss-web-1   1/1       Running   0         1s
```

删除pod，验证ip是否被保存下来：
```
kubectl delete pod -l app=nginx
```

启动一个pod，观察network的状态是否维护了
```
kubectl run -i --tty --image busybox dns-test --restart=Never --rm /bin/sh 
/ # ping ss-web-0.nginx.default.svc.cluster.local
PING ss-web-0.nginx.default.svc.cluster.local (20.0.0.10): 56 data bytes
IP应该不变，是20.0.0.10，状态被维护到了

nslookup ss-web-0.nginx.default.svc.cluster.local
```

## 清理

```
kubectl delete -f ss-web.yaml
kubectl delete -f svc.yaml

```