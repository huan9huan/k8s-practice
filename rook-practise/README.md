# learn how to use rook to manage the ceph block

# refer
https://rook.github.io/docs/rook/master/ceph-quickstart.html

https://rook.github.io/docs/rook/master/block.html


## step 1: create cluster

```
$ kubectl create -f cluster.yaml
```

## step 2: create storage class

```
kubectl create -f storageclass.yaml
```

## step 3: play

```
kubectl create -f mysql.yaml
kubectl create -f wordpress.yaml
```

verify
```
kubectl get pvc
kubectl get svc wordpress
```