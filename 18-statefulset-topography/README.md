## 学习stateful的拓扑结构

### link https://time.geekbang.org/column/article/41017

## 学习目的：
> 理解headless service的DNS的构造, 并验证如何在stateful set中创建这种serivce
> 理解statefulset如何使用headless保持pod的拓扑结构

### headless service的dns构造的实践

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

## 进一步的创建一个临时性的pod，然后执行nslookup观察dns的情况
$ kubectl run -i --tty --image busybox dns-test --restart=Never --rm /bin/sh 

```