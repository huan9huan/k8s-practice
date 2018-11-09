
## 常见的pi上的命令

### 创建一个namespace:pi
```
$ kubectl apply -f ./pi-ns.yaml 
$ kubectl get ns
NAME               STATUS   AGE
default            Active   27d
kube-public        Active   27d
kube-system        Active   27d
pi                 Active   3m56s

```

### 加入一个label到pi上
```
kubectl label nodes yixin9001 device=pi
```

## run pi pod
```
$ kubectl apply -f pi-pod.yaml
$ kubectl get pods -n pi -o wide
$ kubectl describe pod pi-pod -n pi
```

### debug
```
journalctl -u kubelet -f
```

### pi上删除node上的k8s连接
```
sudo rm -fr /etc/kubernetes/
sudo service kubelet stop
```