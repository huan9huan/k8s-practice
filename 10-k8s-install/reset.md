## reset kubeadm环境
```
$ kubeadm reset -f
$ sudo rm -fr $HOME/.kube
$ sudo rm -fr /etc/kubernetes/
```

# remove the cached images
```
for img in `docker images |grep k8s.gcr.io|awk '//{print $3}'`;do docker rmi -f ${img};done
```

## remove weavenet
```
weave reset
rm -f /opt/cni/bin/weave-*
```

## remove rook datadir
```
rm -fr /var/lib/rook
```