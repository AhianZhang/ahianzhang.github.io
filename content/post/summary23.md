---
title: "异地机器组网方案"
date: 2023-05-23T10:33:35+08:00
tags: ["网络"]
categories: ["总结"]
---

最近要自己做一些项目，需要用到 k8s，自己单机搭建比较耗费性能，于是就利用其他资源：一个台式机、一台笔记本、一台阿里云 ECS，那怎么将这三台机器组到一个局域网内呢？

通过调研，发现 [tailscale](https://tailscale.com/) 似乎满足方案，使用开源的 [WireGuard](https://www.wireguard.com/) 协议，能够 p2p 访问，但它是一个商业产品，有连接数的限制，如果个人开发是没什么问题的，但是以后连接的设备一多就需要交钱了。那有没有不收费的？有，[headscale](https://github.com/juanfont/headscale)，这个是一个免费开源产品，实现了一个三方的 tailscale 控制服务器，这样连接数的限制就可控了，客户端依然使用 tailscale。下面看一下操作步骤（均在 root 权限下操作）。

机器配置：

ecs 1c2g



# 安装控制服务器(control server)

这个控终端最好装在不容易断电、重启的机器上，那 ecs 是最佳的选择

## 下载

```shell
# 下载服务端
wget --output-document=/usr/local/bin/headscale \
https://github.com/juanfont/headscale/releases/download/v0.22.3/headscale_0.22.3_linux_amd64
# 设置执行权限
chmod +x /usr/local/bin/headscale
```

## 配置环境

```shell
#创建配置目录：
mkdir -p /etc/headscale

#创建目录用来存储数据与证书：
mkdir -p /var/lib/headscale
#创建目录用来存 so 文件
mkdir -p /var/run/headscale/

#创建空的 SQLite 数据库文件：
touch /var/lib/headscale/db.sqlite

touch /var/run/headscale/headscale.sock



#　从example 创建 Headscale 配置文件：
wget https://github.com/juanfont/headscale/raw/main/config-example.yaml -O /etc/headscale/config.yaml
```

## 修改配置文件

进入到 config.yaml 中修改以下几个点：

- server_url: 设置为``http://<公网 ip>:8080`` 这个要看你当前机器的公网 IP
- randomize_client_port：设为 true

## 创建启动服务

```shell
vi /etc/systemd/system/headscale.service
```

粘贴下面命令

```shell
# /etc/systemd/system/headscale.service
[Unit]
Description=headscale controller
After=syslog.target
After=network.target

[Service]
Type=simple
User=headscale
Group=headscale
ExecStart=/usr/local/bin/headscale serve
Restart=always
RestartSec=5

# Optional security enhancements
NoNewPrivileges=yes
PrivateTmp=yes
ProtectSystem=strict
ProtectHome=yes
ReadWritePaths=/var/lib/headscale /var/run/headscale
AmbientCapabilities=CAP_NET_BIND_SERVICE
RuntimeDirectory=headscale

[Install]
WantedBy=multi-user.target
```

## 创建用户

```shell
# 创建 headscale 用户：

useradd headscale -d /home/headscale -m

# 修改 /var/lib/headscale 目录的 owner：

chown -R headscale:headscale /var/lib/headscale

```

## 启动

### 启动测试

使用 ``headscale serve`` 测试能否正常运行，如果不行根据报错信息修改问题，确认无误后使用 systemctl 启动

### 使用 systemctl 后台运行

```shell
#　Reload SystemD 以加载新的配置文件：

systemctl daemon-reload

#启动 Headscale 服务并设置开机自启：

systemctl enable --now headscale

#　查看运行状态：

systemctl status headscale
```

## 验证

```shell
#　查看运行状态：

systemctl status headscale

#　查看占用端口：

ss -tulnp|grep headscale
```

## 操作

headscale 中有个 namespace 的概念，用来区分不同用户的组网，之间不能互通，类似于多租户的概念

```shell
# 创建一个 namespace，以便后续客户端接入，例如：
headscale namespaces create default

# 查看命名空间：

headscale namespaces list
```



# 安装 MacOS 客户端

访问控制服务中配置的公网 ip 地址，<公网 ip>:8080/apple，会出现一个页面。

找到 [macOS - Standalone Client ](https://pkgs.tailscale.com/stable/#macos)客户端并下载，这是一个 tailscale 的客户端，**需要本机授权才能访问**。

打开后会在顶部菜单栏中出现一个图标![image-20230523111148146](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2023-05-23-031222.png)

## 配置

打开终端，执行下面命令

```shell
defaults write io.tailscale.ipn.macsys ControlURL http://<你的公网ip>:8080
```

此时需要登录 tailscale，按住 alt 点击顶部的 tailscale 图标，会出现 debug 按钮![image-20230523111842985](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2023-05-23-031845.png)

点击 debug-> login，将你的控制服务器地址填进去，<公网 ip>:8080 并登录。

## 认证

此时浏览器会弹出一个页面

![image-20230523112105457](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2023-05-23-032108.png)

拷贝并粘贴到控制服务器所在的机器，我这里是 ecs，在 ecs 终端执行

```shell
headscale nodes register --user 配置的namespace --key nodekey:fcb4522440ce686210
```

如果成功的话会有下面提示：

```shell
Machine 你的namespace registered
```

使用下面命令查看当前网络加入的节点

```sh
headscale node list
```



# 安装 Linux 客户端

从 https://pkgs.tailscale.com/stable/ 找到你系统对应的版本

![image-20230523112734307](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2023-05-23-032737.png)

我的是 centos，直接粘贴官方文档上的命令就可以了。

## 配置

启动

```shell
# 启动 tailscaled.service 并设置开机自启：
systemctl enable --now tailscaled
# 查看服务状态：
systemctl status tailscaled
```

连接

```shell
tailscale up --login-server=http://<公网 ip>:8080 --accept-routes=true --accept-dns=false
# 如果申请为虚拟网关，需要使用下面命令
#tailscale up --login-server=http://<公网 ip>:8080 --accept-routes=true --accept-dns=false --advertise-routes=<当前节点 ip>/24
# 打开转发
echo 'net.ipv4.ip_forward = 1' | tee /etc/sysctl.d/ipforwarding.conf
echo 'net.ipv6.conf.all.forwarding = 1' | tee -a /etc/sysctl.d/ipforwarding.conf
sysctl -p /etc/sysctl.d/ipforwarding.conf
```

此时控制台会出现一个访问地址，拷贝出来用浏览器打开，会得到一串和上面 macos 认证一样命令，可以参考 macos 认证部分



# 验证

此时局域网内有两个节点能够相互访问，可以使用 下面命令查询当前网段下的节点信息

```shell
headscale node list
```

![image-20230523113814723](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2023-05-23-033816.png)

我们可以使用 ping 命令相互 ping 一下，看是否能互通。

![image-20230523113906985](https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2023-05-23-033909.png)
