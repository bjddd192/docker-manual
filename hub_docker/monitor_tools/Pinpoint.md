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
git checkout 1.8.5
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

## PINPOINT采集不到数据

pinpoint-agent配置文件pinpoint.config默认配置profiler.sampling.rate=20，即仅采样5%的事务，每20次请求只收集显示1次请求数据。

```sh
#1 out of n transactions will be sampled where n is the rate. (20: 5%)
profiler.sampling.rate=20
```

如果将此选项设置为1，则代理将每1个事务（100％采样）跟踪一次，如果将其设置为10，则每10个事务（10％采样）跟踪一次。

## 监控数据清理

### 清理失效的 agentID

```sh
http://10.234.6.87:8079/admin/removeInactiveAgents.pinpoint?password=admin
# durationDays 默认 30，必须大于 30 天
http://10.234.6.87:8079/admin/removeInactiveAgents.pinpoint?password=admin&durationDays=30
# 按 agentId 清除数据
http://10.234.6.87:8079/admin/removeAgentId.pinpoint?applicationName=petrel-priv-api&agentId=0e9bdf1e7818&password=admin
# 按 applicationName 清除数据（注意：正常的 agentid 也会清除，需重启应用）
http://10.234.6.87:8079/admin/removeApplicationName.pinpoint?applicationName=petrel-priv-api&password=admin
```

[How to clean up useless pinpoint agentID](https://github.com/naver/pinpoint/issues/6064)

### 清理hbase

数据默认保存 60天。

[pinpoint 修改 hbase 表 TTL 值](https://cloud.tencent.com/developer/article/1423933)

```sh
docker exec -it pinpoint-hbase bash

# 查找出数据大的 hbase 表
cd /home/pinpoint/hbase/data/default
du -sh *
du -h | grep G

# 进入 hbase 修改表 ttl
cd /opt/hbase/hbase-1.2.6/bin
./hbase shell
> list
> describe 'TraceV2'
> disable 'TraceV2'
> alter 'TraceV2' , {NAME=>'S',TTL=>'1209600'}
> enable 'TraceV2'
> describe  'TraceV2'
> major_compact  'TraceV2'
> describe  'ApplicationTraceIndex'
> disable 'ApplicationTraceIndex'
> alter 'ApplicationTraceIndex' , {NAME=>'I',TTL=>'1209600'}
> enable 'ApplicationTraceIndex'
> describe  'ApplicationTraceIndex'
> major_compact  'ApplicationTraceIndex'
> describe  'AgentStatV2'
> disable 'AgentStatV2'
> alter 'AgentStatV2' , {NAME=>'S',TTL=>'1209600'}
> enable 'AgentStatV2'
> describe  'AgentStatV2'
> major_compact  'AgentStatV2'
> 
> # 开发环境执行（清理多余容器id，未成功，清除了agent，数据却不再上来，需要重启应用，不可取）
> list
> truncate 'AgentInfo'
> truncate 'AgentEvent'
> truncate 'AgentLifeCycle'
> truncate 'AgentStatV2'
> count 'AgentInfo'
> scan 'AgentInfo',{LIMIT => 10}
```

## 开启告警

Pinpoint Web会定期检查应用程序的状态，并在满足某些预配置条件（规则）的情况下触发警报。

这些条件（默认情况下）由Web模块中的后台批处理过程每3分钟检查一次（默认情况下），使用的是最后5分钟的数据。一旦满足条件，批处理过程就会向注册到用户组的用户发送短信/电子邮件。

> 如果每次超过阈值时都发送电子邮件/短信，我们认为该警报消息是垃圾邮件。
因此，我们决定逐渐提高警报的传输频率。
例）如果连续发生警报，​​传输频率将增加两倍。3分钟-> 6分钟-> 12分钟-> 24分钟

测试工具，使用 `pinpoint-quickstart` 提供的 web 界面工具，如：http://10.0.43.57:8000/

[I can't receive mail ](https://github.com/naver/pinpoint-docker/issues/18)

[email alarm cannot be triggered with docker deployment](https://github.com/naver/pinpoint/issues/6082)

当前docker版本不支持告警，需等 1.9 版本。

```sh

```

## 参考资料

[docker部署pinpoint，监控docker中的Springboot项目](https://blog.csdn.net/tianyaleixiaowu/article/details/78727050)

[Pinpoint 安装部署](https://www.cnblogs.com/yyhh/p/6106472.html)

[分布式跟踪工具Pinpoint技术入门](https://blog.csdn.net/heyeqingquan/article/details/74456591)

[Pinpoint 分布式请求跟踪系统的搭建](https://segmentfault.com/a/1190000011290541)

[Dapper，大规模分布式系统的跟踪系统](http://bigbully.github.io/Dapper-translation/)

[Pinpoint 安装部署](https://www.cnblogs.com/yyhh/p/6106472.html)

[项目系统监控](https://my.oschina.net/u/3084514/blog/1624907)

[docker-compose部署pinpoint开启email报警功能](https://cloud.tencent.com/developer/article/1423177)

[Pinpoint扩展报警](https://blog.csdn.net/xvshu/article/details/79814549)

[pinpoint-docker开启邮件报警和集成钉钉报警推送](https://juejin.im/post/5ca4ac7d51882543b81adf47)

[Pinpoint 1.8.1-RC1源码编译部署及扩展告警](https://www.gaoyaqiu.com/post/pinpoint/pinpoint-source-compile-alarm/)

[pinpoint1.8.5安装及使用指南](https://www.cnblogs.com/luozhiyun/p/11664534.html)

[Kubernetes 微服务全链路监控Pinpoint（APM）](https://www.liuyalei.top/1692.html)

[Whether pinpoint provide public API?](https://github.com/naver/pinpoint/issues/3245)
