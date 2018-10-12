# 理解job对象

## link: https://time.geekbang.org/column/article/41607

## 学习目的
> 用案例理解job基本用法，错误重试机制
> 理解job-controller的控制器运行模式
> CronJob控制器的结构
> 理解job的常见的部署和使用方式

## 一个计算pi的job
```
$ kubectl create -f ./pi.yaml 

$ kubectl describe jobs/pi

...
Events:
  Type    Reason            Age   From            Message
  ----    ------            ----  ----            -------
  Normal  SuccessfulCreate  5m    job-controller  Created pod: pi-lmqhb
...

$ kubectl log pi-lmqhb

$ kubectl delete -f ./pi.yaml 
```

## 尝试使用cron job
```
$ kubectl create -f ./cronjob.yaml
$ kubectl describe cronjobs/cron-pi
...
Events:
  Type    Reason            Age   From                Message
  ----    ------            ----  ----                -------
  Normal  SuccessfulCreate  22s   cronjob-controller  Created job cron-pi-1539309000
  Normal  SawCompletedJob   12s   cronjob-controller  Saw completed job: cron-pi-1539309000
...

$ kubectl describe job cron-pi-1539309000
...
Events:
  Type    Reason            Age   From            Message
  ----    ------            ----  ----            -------
  Normal  SuccessfulCreate  1m    job-controller  Created pod: cron-pi-1539309000-r876b
...


kubectl get cronjobs/cron-pi -w
NAME      SCHEDULE      SUSPEND   ACTIVE    LAST SCHEDULE   AGE
cron-pi   */1 * * * *   False     0         <none>          21s
cron-pi   */1 * * * *   False     1         1s        30s
cron-pi   */1 * * * *   False     0         11s       40s


$ kubectl delete -f ./cronjob.yaml
```

一些关键的注意点：  
> cronjob controller会每分钟生成一个job，每个job有会生成一个pod，于是pod就越来越多

