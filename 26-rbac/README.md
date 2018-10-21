# 理解Role Based Access Control

### link https://time.geekbang.org/column/article/42154

## 学习目的
> 尝试建议一个对default的ns做read的role

## 练习：尝试建议一个对default的ns做read的role

### step 1: 准备一个ns和此ns下的deployment
```
$ kubectl create -f ./dev-ns.yaml
$ kubectl get ns
NAME          STATUS    AGE
default       Active    33d
dev           Active    2m
...

$ kubectl create -f ./dev-ds.yaml
$ kubectl get rs -n dev 
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-75675f5897   1         1         1         1m
```

### step 2: 在dev的ns下创建一个readonly的role，并assign给default sa
```
kubectl create -f ./ro-binding-dev.yaml
kubectl get rolebindings -n dev
```

### step 3: 在所有的ns下创建一个readonly的role，并assign给所有default sa
```
kubectl create -f ./ro-binding-all.yaml

kubectl get clusterrolebindings -o json | jq -r '.items[] | select(.subjects[0].kind=="User") | select(.subjects[0].name=="system.serviceaccount:default")'
```
TODO: 如何list出来所有的用户下