
## 常见的pi上的命令

### 加入一个label到pi上
```
kubectl label nodes yixin9001 device=pi
```

### debug
```
journalctl -u kubelet
```