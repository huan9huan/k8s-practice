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

如下的脚本在pi上运行
```
#!/bin/sh

# This installs the base instructions up to the point of joining / creating a cluster

curl -sSL get.docker.com | sh && \
  sudo usermod pi -aG docker

sudo dphys-swapfile swapoff && \
  sudo dphys-swapfile uninstall && \
  sudo update-rc.d dphys-swapfile remove

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

echo "deb https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list 

sudo apt-get update -q && \
sudo apt-get install -qy kubeadm

echo Adding " cgroup_enable=cpuset cgroup_enable=memory" to /boot/cmdline.txt

sudo cp /boot/cmdline.txt /boot/cmdline_backup.txt
orig="$(head -n1 /boot/cmdline.txt) cgroup_enable=cpuset cgroup_enable=memory"

```

## update the damon json
```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://877id5g1.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## install sources

```
vim /etc/apt/sources.list
vim /etc/apt/sources.list.d/raspi.list 

version=1.12.1
kubeadm config images list --kubernetes-version=v${version} --feature-gates CoreDNS=false

sudo apt-get install -y kubelet=${version}-00 kubeadm=${version}-00 kubectl=${version}-00
sudo apt-mark hold kubelet=${version}-00 kubeadm=${version}-00 kubectl=${version}-00
sudo systemctl enable kubelet && sudo systemctl start kubelet

kubeadm config images list --kubernetes-version=v${version} --feature-gates CoreDNS=false
```

## join into cluster
```
sudo rm /etc/kubernetes/bootstrap-kubelet.conf
sudo rm /etc/kubernetes/pki/ca.crt 

sudo kubeadm join -v 2 139.196.145.43:6443 --token kbxdut.s9wx39i7na5k0mu9 --discovery-token-ca-cert-hash sha256:cc05824d2e94e5699fd9164f638d88d027167e94842854e6d5fba9185fb6acc5
```


sudo cat <<EOT >/etc/cni/net.d/10-weave.conf 
{
    "name": "weave",
    "type": "weave-net",
    "hairpinMode": true
}
EOT

