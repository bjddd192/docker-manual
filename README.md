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

