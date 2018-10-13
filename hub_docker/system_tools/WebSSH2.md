# WebSSH2

## 简介

与 GateOne 类似，也是一款使用 HTML5 技术编写的网页版 SSH 终端模拟器。

特点是更轻量，可以方便地抓取服务器操作产生的日志。

更多内容请参考：

[webssh2官网](https://www.npmjs.com/package/webssh2)

## 官方镜像

[psharkey/webssh2](https://hub.docker.com/r/psharkey/webssh2/)

## 启动命令

```sh
docker run -d --name webssh2 -p 2222:2222 --restart=always \
  psharkey/webssh2
```

## 服务使用

访问：http://172.20.32.36:2222/ssh/host/172.20.32.47 即可。

## 参考资料

[一个可以在浏览器上运行的SSH客户端：WebSSH2安装教程](https://www.moerats.com/archives/467/)
