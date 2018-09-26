# pod - 1 学习笔记 - 

## 链接：https://time.geekbang.org/column/article/40366

```
kubectl get pods simple-pod -o wide
```

## task 1: 不拉取image的时候的效果
pod中改变 imagePullPolicy字段
```
  containers:
    - name: shell
      imagePullPolicy: IfNotPresent
      image: busybox
      tty: true
      stdin: true

    - name: nginx
      imagePullPolicy: IfNotPresent
      image: nginx:1.7.9
```

`kubectl describe pods simple-pod`

注意到events中出现：

```
Container image "busybox" already present on machine
Container image "nginx:1.7.9" already present on machine
```

## task 2: 理解pod级别的设置
pod: pod-tty.yaml
其中增加了hostPID等pod级别的设置

```
# delete if need
# kubectl delete -f ./pod-tty.yaml

kubectl create -f ./pod-tty.yaml
kubectl describe pod pod-tty
# wait for a while
kubectl attach -it pod-tty -c shell

# run `ps ax` 应该看到很多的pid
```

**todo: 单独使用`shareProcessNamespace: true`无法得到pod中的pid**
