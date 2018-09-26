# pod - 1 学习笔记 - 

## 链接：https://time.geekbang.org/column/article/40366

```
kubectl get pods simple-pod -o wide
```

## Task 1: 不拉取image的时候的效果
pod中改变 imagePullPolicy字段
```
  containers:
    - name: shell
      imagePullPolicy: IfNotPresent
      image: busybox
      tty: true
      stdin: true
```

`kubectl describe pods simple-pod`

注意到events中出现：

```
Container image "busybox" already present on machine
Container image "nginx:1.7.9" already present on machine
```
