# 简介

```sh
docker run -it --rm hub.wonhigh.cn/basic/alpine-java:8_jdk date
docker run -it --rm hub.wonhigh.cn/basic/alpine-java:7_jdk date
docker run -it --rm anapsix/alpine-java:7_jdk bash

# 在容器内取容器ID
cat /proc/self/cgroup | grep pids | awk -F '/' '{print $NF}' | cut -c1-12

# 如何从容器内部获取容器ID
cat /proc/self/mountinfo
或
cat /proc/self/mounts
或
cat /proc/self/cgroup
```

https://github.com/bcicen/ctop

[Docker 官网](https://www.docker.com)

[what-container](https://www.docker.com/resources/what-container)

[容器如“衣服”，而虚拟机却是“房子”](http://virtual.51cto.com/art/201803/568648.htm)

[Docker | 第五章：构建自定义镜像](https://hk.saowen.com/a/7e1f47946c6d437bd4e8cfcf91b06a4ccd51111cfc9d224aff01f22abd891f76)

[10张图带你深入理解Docker容器和镜像](http://dockone.io/article/783)

[docker mtu介绍](https://www.dazhuanlan.com/2019/12/09/5dee42c8950ed/)


镜像加速技巧：
quay.io 的可以用 ustc 的
quay.mirrors.ustc.edu.cn/kubernetes-ingress-controller/nginx-ingress-controller:0.22.0

[Azure 镜像代理](https://www.chenshaowen.com/blog/developing-tips-18.html#1-azure-%E9%95%9C%E5%83%8F%E4%BB%A3%E7%90%86)

docker stack

[Docker 网络代理设置](https://my.oschina.net/styshoo/blog/841308)
