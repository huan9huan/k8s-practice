# 理解容器网络

## link https://time.geekbang.org/column/article/64948

### 核心概念:

# task1 ：两个container如何互通的

```
# 第一个container
$ docker run -it --name b1 busybox sh
$ ifconfig
发现ip是 192.168.0.4
$ route
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         192.168.0.1     0.0.0.0         UG    0      0        0 eth0
192.168.0.0     *               255.255.240.0   U     0      0        0 eth0

# 第二个container
$ docker run -it --name b2 busybox sh
$ ifconfig
发现ip是 192.168.0.5
$ ping 192.168.0.4

```
原因：
> b2 ping 192.168.0.4(b1)的时候，先匹配了route中的第二条规则
> 第二条规则中，发现Gateway是*，表示直通，直接使用eth0寻找mac地址，从而能够调用二层网络
> eth0是一个Veth Pair，插在docker0中，arp请求会被docker0完成，docker0会把对应的b1的mac地址传给b2


## task 2: 如何定位Veth的关系

```
# run in host
$ brctl show
```