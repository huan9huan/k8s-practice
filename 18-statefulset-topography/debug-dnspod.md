## dsn lookup 无法运行，debug找原因

**思路：先找到pod的name，然后得到container的名称，然后得到log**

```
$ kubectl get pods  -n kube-system

NAME                                         READY     STATUS             RESTARTS   AGE
etcd-docker-for-desktop                      1/1       Running            0          19d
kube-apiserver-docker-for-desktop            1/1       Running            0          19d
kube-controller-manager-docker-for-desktop   1/1       Running            2          19d
kube-dns-86f4d74b45-28xdg                    3/3       Running            0          19d
kube-proxy-44dtt                             1/1       Running            0          19d
kube-scheduler-docker-for-desktop            1/1       Running            0          19d
kubernetes-dashboard-7b9c7bc8c9-r7tjn        0/1       ImagePullBackOff   0          19d


$ kubectl logs -f kube-dns-86f4d74b45-28xdg kubedns -n kube-system
```

结果得到了一个log
```
I0926 03:19:44.324697       1 server.go:126] Status HTTP port 8081
I1006 03:47:56.022862       1 dns.go:555] Could not find endpoints for service "nginx" in namespace "default". DNS records will be created once endpoints show up.
```
而且这个log是发生在headless service的创建的时刻 `kubectl create -f svc.yaml`
如果是去掉   `clusterIP: None` 就不会有这个错误