## Service/DNS/服务发现

## link: https://time.geekbang.org/column/article/68636

## 学习目的
1. service的内部机制是什么
2. ipvs是怎么回事

### task 1: 使用hostname的deployment理解service的实现
```
kubectl create -f ./hostname.yaml
```