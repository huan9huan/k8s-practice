# 在digital ocean上安装k8s cluster

### step 1: install master (bot@d2)
```
sudo kubeadm init

## 配置kubectl
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

```

### step 2: install node in (root@v1)
```
# 需先停掉老的k8s
service kubelet status
service kubelet stop
rm -fr /etc/kubernetes/

sudo kubeadm join 138.68.29.180:6443 --token qcowhi.fagac0gdsjygtvk9 --discovery-token-ca-cert-hash sha256:79fa975ddfa5ff69c6930d3d757b3fce2125761ab7b5c8d6b6f922d4fea2892e
```

### step 3: install network

master
```

$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')&env.IPALLOC_RANGE=172.30.0.0/16"

kubectl get pods --all-namespaces -o wide

kubectl describe pod coredns-78fcdf6894-9qtrc -n kube-system

kubectl logs -f coredns-78fcdf6894-9qtrc -n kube-system

sudo service kubelet restart
```

## TroubleShots

### coredns的pod启动不起来
可能是weavenet的iprange有问题，和宿主机的网络ip重叠了
使用 `kubectl logs -f coredns-78fcdf6894-9qtrc -n kube-system` 去确认
解决方法参见 https://github.com/weaveworks/weave/blob/master/site/tasks/ipam/configuring-weave.md

