## 理解经典的负载均衡在k8s中如何表达

### link https://time.geekbang.org/column/article/40906

## 学习目的
> 验证水平扩展和收缩如何实现的    
> 进一步感受控制器模式的设计规则
> 体会直接使用kubectl改动yaml的配置


## 使用命令行操作deployment的运行时属性
基本流程：
> 1. 创建2个nginx的pod，理解pod-template-hash，学习观察status,rs
> 2. 用命令行改动image值，然后观察热更新的过程
> 3. 改动一个image，模拟一次错误的升级，然后学习如何做版本回滚

```
# 创建一个基础的deployment对象
$ kubectl create -f base.yaml --record

## 打印deployment的属性
$ kubectl get deployment

## 实时查看D状态数据，如果ready，则退出
$ kubectl rollout status deployment/nginx-deployment

## 查看rs的状态
$ kubectl get rs
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-75675f5897   2         2         2         3m
## 注意理解pod-template-hash字段：75675f5897


## 练习改动属性，并观察效果
###  在一个窗口中执行：
$ while true; do kubectl rollout status deployment/nginx-deployment; sleep 1; done
### 另一个窗口中执行
$ kubectl edit deployment/nginx-deployment
... 
    spec:
      containers:
      - name: nginx
        image: nginx:1.9.1 # 1.7.9 -> 1.9.1
        ports:
        - containerPort: 80
...

## 打印出
deployment "nginx-deployment" successfully rolled out
Waiting for deployment spec update to be observed...
Waiting for deployment spec update to be observed...
Waiting for deployment spec update to be observed...
Waiting for deployment "nginx-deployment" rollout to finish: 0 out of 2 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 1 out of 2 new replicas have been updated...
^[[A
Waiting for deployment "nginx-deployment" rollout to finish: 1 out of 2 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 1 out of 2 new replicas have been updated...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "nginx-deployment" rollout to finish: 1 old replicas are pending termination...


## 模拟一次发布失败的情况（设定错误的image，然后rs滚定更新失败）
$ kubectl set image deployment/nginx-deployment nginx=nginx:1.91
$ kubectl get rs

NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-595696685f   1         1         0         8s
nginx-deployment-75675f5897   0         0         0         13m
nginx-deployment-c4747d96c    2         2         2         6m
## 注意这里新的pod-temp-hash是595696685f，但是ready是0

## 然后回滚
$ kubectl rollout undo deployment/nginx-deployment
$ kubectl get rs
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-595696685f   0         0         0         2m
nginx-deployment-75675f5897   0         0         0         16m
nginx-deployment-c4747d96c    2         2         2         8m
$ kubectl rollout history deployment/nginx-deployment

```
