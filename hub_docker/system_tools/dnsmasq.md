# dnsmasq

## 简介

GateOne 是一款使用 HTML5 技术编写的网页版 SSH 终端模拟器，非常高大尚，它有以下特点：

1. 基于现代的 HTML5 技术，无需任何浏览器插件。
2. 支持多个 SSH 进程。
3. 可以嵌入到其他任意应用程序中。
4. 支持使用 JavaScript，Python 甚至纯 CSS 编写的插件。
5. 支持 SSH 进程副本，打开多个进程而无需重复输入密码。
6. 支持各种服务器端的日志功能，支持 Keberos-based 单点登录甚至活动目录。
7. 支持操作日志记录，具有操作记录回放功能。

综上所述，GateOne 可以作为一种堡垒机的开源解决方案。

## 官方镜像

[andyshinn/dnsmasq](https://hub.docker.com/r/andyshinn/dnsmasq)

[我的镜像](https://hub.docker.com/r/bjddd192/dnsmasq)

## 启动命令

```sh
mkdir -p /data/docker_volumn/dnsmasq

tee > /data/docker_volumn/dnsmasq/resolv.dnsmasq <<'EOF'
# 配置上外部的 dns 服务器地址
nameserver 172.20.32.132
nameserver 202.102.213.68
nameserver 114.114.114.114
EOF

tee > /data/docker_volumn/dnsmasq/dnsmasqhosts <<'EOF'
# 配置上内部的 dns 服务解析规则，在使用的过程中会经常需要配置此文件
172.20.32.36     nexus.belle.cn
EOF

tee > /data/docker_volumn/dnsmasq/dnsmasq.conf <<'EOF'
# dnsmasq 的配置文件
resolv-file=/etc/resolv.dnsmasq
addn-hosts=/etc/dnsmasqhosts
EOF

docker stop dnsmasq && docker rm dnsmasq

docker run -d --name dnsmasq -p 53:53/tcp -p 53:53/udp \
  --cap-add=NET_ADMIN --restart=always \
  -v /data/docker_volumn/dnsmasq/dnsmasq.conf:/etc/dnsmasq.conf \
  -v /data/docker_volumn/dnsmasq/dnsmasqhosts:/etc/dnsmasqhosts \
  -v /data/docker_volumn/dnsmasq/resolv.dnsmasq:/etc/resolv.dnsmasq \
  bjddd192/dnsmasq:2.78
```

## 防火墙设置

如服务器启用了防火墙，需要增加规则：

```sh
firewall-cmd --direct --permanent --add-rule ipv4 filter INPUT 0 -p tcp --dport 53 -j ACCEPT
firewall-cmd --direct --permanent --add-rule ipv4 filter INPUT 0 -p udp --dport 53 -j ACCEPT
firewall-cmd --reload
```

## 客户端使用

### linux

找到 `/etc/sysconfig/network-scripts/` 下的物理网卡文件增加或修改 DNS 配置，然后重启网络服务，如下：

```sh
vi /etc/sysconfig/network-scripts/ifcfg-eth0 
NAME="eth0"
HWADDR="00:50:56:87:58:06"
ONBOOT=yes
NETBOOT=yes
UUID="44721538-1fbe-4d73-98dc-bd6dafff03c6"
IPV6INIT=yes
BOOTPROTO=static
TYPE=Ethernet
IPADDR=10.234.6.83
NETMASK=255.255.255.0
GATEWAY=10.234.6.1
DNS1=10.0.43.27

# 重启网络服务
systemctl restart network
```

### windows

直接到网络中添加此 dns 服务，注意应该放置到最前面。

## 参考资料

[Docker下搭建DNS服务器dnsmasq](https://www.tuicool.com/articles/Nfiqyyj)

[Dnsmasq安装与配置](http://www.cnblogs.com/kevinchou/p/5844091.html)
