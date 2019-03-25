# Pinpoint

Pinpoint 是世界领先的开源应用监控解决方案 - 受到全球数百万用户的信赖。它支持并帮助您一目了然地了解您的应用程序，并允许您构建世界一流的高质量软件。

Pinpoint 是开源在 github 上的一款 APM 监控工具，它是用 Java 编写的，用于大规模分布式系统监控。它对性能的影响最小（只增加约 3％ 资源利用率），安装 agent 是无侵入式的，只需要在被测试的 Tomcat 中加上 3 句话，打下探针，就可以监控整套程序了。

[官网](https://github.com/naver/pinpoint-docker)

[Hub官方](https://hub.docker.com/u/pinpointdocker/)

## 容器化安装

```sh
git clone https://github.com/naver/pinpoint-docker.git
cd pinpoint-docker
# 查看 tag 版本
git tag
# 切换 tag 版本
git checkout 1.8.0
# 这时候 git 可能会提示你当前处于一个“detached HEAD" 状态。
# 因为 tag 相当于是一个快照，是不能更改它的代码的。
# 如果要在 tag 代码的基础上做修改，你需要一个分支： 
git checkout -b branch_name tag_name
# 查看当前分支
git branch -a

# 检查调整 docker-compose.yaml 文件的网络模式等
# 检查配置 .env 文件参数
docker-compose pull && docker-compose up -d

# 调试基础 jdk
docker run -it --rm -e AGENT_DEBUG_LEVEL=DEBUG -e COLLECTOR_IP=10.0.43.25 hub.wonhigh.cn/basic/alpine-java:8_jdk_pinpoint_agent bash
```

仓库比较大，镜像比较多，需要耐心等待。

测试地址：

http://10.0.43.25:8079

http://10.0.43.25:8081

## 参考资料

[docker部署pinpoint，监控docker中的Springboot项目](https://blog.csdn.net/tianyaleixiaowu/article/details/78727050)

[Pinpoint 安装部署](https://www.cnblogs.com/yyhh/p/6106472.html)

[分布式跟踪工具Pinpoint技术入门](https://blog.csdn.net/heyeqingquan/article/details/74456591)

[Pinpoint 分布式请求跟踪系统的搭建](https://segmentfault.com/a/1190000011290541)

[Dapper，大规模分布式系统的跟踪系统](http://bigbully.github.io/Dapper-translation/)

[Pinpoint 安装部署](https://www.cnblogs.com/yyhh/p/6106472.html)

[项目系统监控](https://my.oschina.net/u/3084514/blog/1624907)
