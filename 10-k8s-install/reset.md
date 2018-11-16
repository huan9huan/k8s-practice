## reset kubeadm环境
```
$ sudo kubeadm reset -f
$ sudo rm -fr $HOME/.kube
$ sudo rm -fr /etc/kubernetes/
sudo ip link del cni0
sudo ip link del weave
```

# remove the cached images
```
for img in `docker images |grep k8s.gcr.io|awk '//{print $3}'`;do docker rmi -f ${img};done
```

## remove weavenet
```
weave reset
sudo rm -f /opt/cni/bin/weave-*
sudo ip link del weave
sudo rm -fr /etc/cni/net.d/
```

## remove rook datadir
```
rm -fr /var/lib/rook
```