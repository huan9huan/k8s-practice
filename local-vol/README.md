## learn how to mount local dir into pv

### step 1: 创建一个local 的 storage class, 生成一个pv, 然后在nginx中挂在pvc

```
kubectl delete -f .
kubectl apply -f .
kubectl get pods -w -o wide

kubectl describe svc web-svc
kubectl get svc 
## should get the endpoints
```

### step 2: check if the html existed in local disk
```
cat /var/gshop/data/test.html
```
```
Sat Feb 16 03:53:55 UTC 2019
Sat Feb 16 03:59:54 UTC 2019
```

### check the web works  
kubectl exec ss-web-0 -- ls /var/www/
kubectl exec ss-web-0 -- cat /etc/nginx/conf.d/default.conf

```
kubectl run -i --tty --image odise/busybox-curl dev --restart=Never --rm /bin/sh 

pod# curl web-svc

hello Sat Feb 16 04:14:22 UTC 2019
```
pod内的cluster中访问svc成功, 说明已经完成

### check site in node
```
kubectl port-forward web-ss-0 30008:80
```

在host中
```
node$ curl 127.0.0.1:30008
```

hello Sat Feb 16 04:14:22 UTC 2019

获得了数据

### check in host disk
```
node$ cat /var/gshop/data/index.html 
hello Sat Feb 16 04:14:22 UTC 2019
```
说明已经在本地产生了

