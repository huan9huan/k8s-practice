## reset kubeadm环境

```
for img in `docker images |grep k8s.gcr.io|awk '//{print $3}'`;do docker rmi -f ${img};done
```