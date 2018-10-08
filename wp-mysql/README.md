## 使用k8s部署wp和mysql

## follow link https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/

## step 1: mysql secret
```
kubectl create secret generic mysql-pass --from-literal=password=123456

kubectl get secrets

mysql-pass            Opaque                                1         9ss
```

## step 2: deploy mysql server

创建mysql pod，使用hostpath作为pv
```
kubectl create -f mysql-deployment.yaml
kubectl get pvc
kubectl get pods
```

debug 命令：
```

kubectl run -i --tty --image mysql:5.6 mysql-test --restart=Never --rm /bin/sh 

kubectl exec wordpress-mysql-bcc89f687-djw2f -- sh -c 'cat /etc/hosts'

10.1.0.47 wordpress-mysql-bcc89f687-djw2f

mysql -h wordpress-mysql-bcc89f687-djw2f.wordpress-mysql.default.svc.cluster.local -u root -p123456
mysql -h 10.1.0.47 -u root -p123456
```

## step 3: deploy wordpress
```
kubectl create -f ./wp-deployment.yaml 
kubectl get pvc

kubectl get services wordpress
```

然后访问 localhost:80 即可