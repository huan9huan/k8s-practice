# 记录如何在pi上安装k8s

## refer https://blog.csdn.net/liukuan73/article/details/83150473 

## 目的：在树莓派上安装k8s，并加入到master中


## step1: 禁用SELinux
永久禁用：
```
sudo vi /etc/selinux/config
SELINUX=permissive
```

## step 2: swap off
```
# 临时命令 - 不需要重启
sudo swapoff -a

# 永久 - 需要重启
sudo update-rc.d dphys-swapfile remove

## 诊断命令
swapon -s

# 当挂掉的时候，可以使用这个命令来让fs可写
mount -w -o remount /
```

## step 3: memory的cgroup功能
```
echo Adding " cgroup_enable=cpuset cgroup_enable=memory" to /boot/cmdline.txt

sudo cp /boot/cmdline.txt /boot/cmdline_backup.txt
orig="$(head -n1 /boot/cmdline.txt) cgroup_enable=cpuset cgroup_enable=memory"
echo $orig | sudo tee /boot/cmdline.txt

echo Please reboot
```

## step 4: install docker
```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://877id5g1.mirror.aliyuncs.com"],
  "bip" : "192.168.0.1/20"
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

```

## step 4: worker node 上安装image镜像（pause+proxy）
```
VERSION=v1.12.1

sudo kubeadm config images list --kubernetes-version=${VERSION} --feature-gates CoreDNS=false

COREDNS_VERSION=1.2.2
PAUSE_VERSION=3.1
MY_REGISTRY=registry.cn-hangzhou.aliyuncs.com/google_containers

## 拉取镜像
docker pull ${MY_REGISTRY}/kube-proxy:${VERSION}
docker pull ${MY_REGISTRY}/pause:${PAUSE_VERSION}

## 添加Tag

docker tag ${MY_REGISTRY}/kube-proxy:${VERSION} k8s.gcr.io/kube-proxy:${VERSION}
docker tag ${MY_REGISTRY}/pause:${PAUSE_VERSION} k8s.gcr.io/pause:${PAUSE_VERSION}
```


## join into cluster
```
sudo rm /etc/kubernetes/bootstrap-kubelet.conf
sudo rm /etc/kubernetes/pki/ca.crt 

sudo kubeadm join -v 2 139.196.145.43:6443 --token kbxdut.s9wx39i7na5k0mu9 --discovery-token-ca-cert-hash sha256:cc05824d2e94e5699fd9164f638d88d027167e94842854e6d5fba9185fb6acc5
```

## Troubleshots
### cni在pi上报错误，找不到cni

方法：`/var/lib/kubelet/kubeadm-flags.env` 去掉 `--network-plugin=cni`
