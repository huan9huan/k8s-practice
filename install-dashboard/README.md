# how to install k8s dashboard

## install and forward port
```
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard.yaml

kubectl port-forward kubernetes-dashboard-669f9bbd46-ndgwn 8443:8443 -n kube-system
```

## get token to login
[open site](https://localhost:8443/)

login:
```
kubectl -n kube-system get secret |grep deployment-controller-token
```