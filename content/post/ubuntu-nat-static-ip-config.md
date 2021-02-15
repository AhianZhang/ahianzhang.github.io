---
title: "VMware 12 NAT网络下配置 ubuntu 16.04 LTS 系统静态 IP"
date: "2018-05-09 20:25:53"
tags:  ["nat"]
categories: ["架构"]
---
镜像是 ubuntu 16.04 server 版，主机是 Windows10 系统，因为要搭建 Hadoop 集群，所以配置一下，将此配置流程在此记录，方便查找。

### 查看主机 IP
```
ipconfig
```

系统分配给虚拟机使用的网卡是 VMnet8，其内容如下：
![TIM截图20180509211132](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2020-07-08-025521.png)

如果有这块网卡则说明出于开启状态

### 查看 VMware 的虚拟网络选项
![TIM截图20180509211424](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2020-07-08-025849.png)

子网 ip 最后以 0 结尾，这个术语叫什么不知道，网关 ip 以 2 结尾
例如： 子网 ip ：192.168.137.0   网关： 192.168.137.2
必须保证其在一个网段以内。

### 修改 Ubuntu 内的配置

#### 查看系统分配的网卡名称
```
 ifconfig -a
```
有的可能是 eth0 有的可能是 ens33，我的是 ens33

#### 配置网卡

```
sudo vi /etc/network/interfaces  
```

```bash
auto ens33
iface ens33 inet static
address 192.168.137.155 #这个ip一般是128~255
network 255.255.255.0
gateway 192.168.137.2
```
![TIM截图20180509212747](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2020-07-08-025905.png)
添加 DNS 解析

```
sudo  vim /etc/resolvconf/resolvconf.d/head
```

```
nameserver 144.144.144.144
nameserver 8.8.8.8
nameserver 1.1.1.1
```

### 重启 network 服务
```
sudo /etc/init.d/networking restart
```
### 重启服务器
```
sudo reboot
```

### 测试
ping www.baidu.com 收到数据，无丢包