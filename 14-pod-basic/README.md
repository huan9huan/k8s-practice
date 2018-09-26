# pod - 1 学习笔记 - 

## 链接：https://time.geekbang.org/column/article/40366

## 学习目的
> 理解pod的核心概念   
> 理解k8s的诊断机制   
> 训练从docker cli到kube cli的过度

## task 1: 不拉取image的时候的效果
pod中改变 imagePullPolicy字段
```
  containers:
    - name: shell
      imagePullPolicy: IfNotPresent
      ...

    - name: nginx
      imagePullPolicy: IfNotPresent
      ....
```

`kubectl describe pods simple-pod`

注意到events中出现：

```
Container image "busybox" already present on machine
Container image "nginx:1.7.9" already present on machine
```

## task 2: 理解pod级别的设置
pod: pod-share.yaml
其中增加了hostPID等pod级别的设置

```
# delete if need
kubectl delete -f ./pod-share.yaml

kubectl create -f ./pod-share.yaml
kubectl describe pod pod-share
# wait for a while
kubectl attach -it pod-share -c shell

# run `ps ax` 应该看到很多的pid
```

Try More:  
> 如果去掉了 `tty: true` 后，会发现pod启动不起来，因为直接关闭了的原因
> 如果去掉了 `stdin: true` 后，会无法交互, 因为stdin被关闭了

> **todo: 直接使用 `shareProcessNamespace: true` 无法得到pod中的pid**
