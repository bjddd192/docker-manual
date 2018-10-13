# GateOne

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

[liftoff/gateone](https://hub.docker.com/r/liftoff/gateone/)

## 启动命令

```sh
docker run -d --name gateone -h gateone -p 12222:8000 --restart=always \
  -e TZ=Asia/Shanghai \
  liftoff/gateone gateone
```

## 服务使用

访问：https://172.20.32.36:12222/ 即可。

快捷访问：https://172.20.32.36:12222/?ssh=ssh://root@172.20.32.47 即可。

## 参考资料

[开源web终端ssh解决方案-gateone简介](http://blog.51cto.com/itnihao/1311506)

[一款非常好用的Web端SSH工具：GateOne安装教程](https://www.moerats.com/archives/582/)

[GateOne —— 高效的WebSSH工具](https://blog.ilemonrain.com/docker/liftoff-gateone.html)

## 扩展阅读

[GateOne配置API认证、SSH自动登录、用户免密登录及Web应用嵌入](https://blog.csdn.net/ZeroChia/article/details/82683002)
