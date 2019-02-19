## learn how to use ingress

### step 1: prepare the nginx controller in k8s
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml
```

## step 2: add nginx nodeport to expose service
```
kubectl apply -f igress-nodeport.yml
kubectl get svc -n ingress-nginx

NAME            TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx   NodePort   10.100.134.229   <none>        80:30598/TCP,443:32202/TCP   46s

IC_IP=192.168.1.6 # 宿主机的地址
IC_HTTPS_PORT=30598 # NodePort 端口

```

## step 2: create service

```
kubectl apply -f ./ingress-cafe.yml

curl --resolve cafe.example.com:$IC_HTTPS_PORT:$IC_IP http://cafe.example.com:$IC_HTTPS_PORT/tea

Server address: 10.1.0.150:80
Server name: tea-58d4697745-5ds4n
Date: 19/Feb/2019:14:59:08 +0000
URI: /tea
Request ID: af1461bb66f91ce84e78f6d81963ddfb

```

## step 3: watch nginx config file
```
kubectl get pods -n ingress-nginx

NAME                                        READY     STATUS    RESTARTS   AGE
nginx-ingress-controller-75cd585bbc-77mlq   1/1       Running   0          29m

kubectl describe pod nginx-ingress-controller-75cd585bbc-77mlq -n ingress-nginx

kubectl exec nginx-ingress-controller-75cd585bbc-77mlq -n  ingress-nginx -- ls /etc/nginx
kubectl exec nginx-ingress-controller-75cd585bbc-77mlq -n  ingress-nginx -- cat /etc/nginx/nginx.conf
```

Notices:

grep tea等可以看到关于/tea的路由信息被配置到了nginx.conf

## conclusion:
总体来说, nginx controller帮你实现了nginx的配置解析, 然后nginx暴露一个公共的service,集中管理conf的配置