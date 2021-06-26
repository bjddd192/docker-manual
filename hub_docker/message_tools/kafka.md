# kafka

## kafka 使用背景

在我们大量使用分布式数据库、分布式计算集群的时候，是否会遇到这样的一些问题：

- 我们想分析下用户行为（pageviews），以便我们设计出更好的广告位
- 我想对用户的搜索关键词进行统计，分析出当前的流行趋势
- 有些数据，存储数据库浪费，直接存储硬盘效率又低 

这些场景都有一个共同点：

**数据是由上游模块产生，使用上游模块的数据计算、统计、分析，这个时候就可以使用消息系统，尤其是分布式消息系统！**

## kafka 定义

kafka 是一个分布式消息系统，由 linkedin 使用 scala 编写，用作 LinkedIn 的活动流（Activity Stream）和运营数据处理管道（Pipeline）的基础。具有高水平扩展和高吞吐量。

## 常用消息组件对比

![消息组件对比图](/images/消息组件对比图.png)

## 关键名词

- broker：kafka 集群包含一个或者多个服务器，服务器就称作 broker。
- producer：负责发布消息到 broker。
- consumer：消费者，从 broker 获取消息。
- topic：发布到 kafka 集群的消息类别。
- partition：每个 topic 划分为多个 partition。
- group：每个 partition 分为多个 group。

## kafka 教程

[Kafka 中文文档 - ApacheCN](http://kafka.apachecn.org/)

[kafka 中文教程](http://www.orchome.com/kafka/index)

[Apache Kafka 教程](https://www.w3cschool.cn/apache_kafka/)

[quickstart](http://kafka.apache.org/quickstart)

[v1.1 broker configs](http://kafka.apache.org/11/documentation.html#brokerconfigs)

## 版本演变

[一文看懂 kafka 消息格式的演变](https://blog.csdn.net/u013256816/article/details/80300225)

[kafka 0.8--0.11各个版本特性预览介绍](http://www.cnblogs.com/intsmaze/p/6709297.html)

[kafka 各个版本差异汇总](https://blog.csdn.net/weixin_34133829/article/details/94659514)

[kafka的版本号与版本演进](https://blog.csdn.net/liuxiao723846/article/details/106020738/)

## 官方镜像

[wurstmeister/zookeeper](https://hub.docker.com/r/wurstmeister/zookeeper)

[wurstmeister/kafka](https://hub.docker.com/r/wurstmeister/kafka)

[sheepkiller/kafka-manager](https://hub.docker.com/r/sheepkiller/kafka-manager)

[fast-data-dev / kafka-lenses-dev (Lenses Box)](https://hub.docker.com/r/landoop/fast-data-dev)

## 编排文件

kafka 1.11 版本：

```yml
version: '2.1'

services:

  zookeeper:
    image: wurstmeister/zookeeper:latest
    container_name: zookeeper
    restart: always
    ports:
    - "2181:2181"
    environment:
    - TZ=Asia/Shanghai
    volumes:
    - /data/docker_volumn/zookeeper/data:/data
    - /data/docker_volumn/zookeeper/datalog:/datalog
    network_mode: bridge
    
  kafka:
    image: wurstmeister/kafka:2.11-1.1.1
    container_name: kafka
    restart: always
    ports:
    - "9092:9092"
    environment:
    - KAFKA_ADVERTISED_HOST_NAME=10.0.43.19
    - KAFKA_ZOOKEEPER_CONNECT=10.0.43.19:2181
    - KAFKA_MESSAGE_MAX_BYTES=2000000
    - KAFKA_AUTO_CREATE_TOPICS_ENABLE=true
    volumes:
    - /data/docker_volumn/kafka/logs:/kafka
    - /var/run/docker.sock:/var/run/docker.sock
    network_mode: bridge

  kafka-manager:
    image: sheepkiller/kafka-manager:latest
    container_name: kafka-manager
    restart: always
    ports:
    - "9000:9000"
    environment:
    - ZK_HOSTS=10.0.43.19:2181
    network_mode: bridge
```

## 读写验证

```sh
# 创建 topics
docker exec -it kafka kafka-topics.sh --create --zookeeper 10.0.43.19:2181 --replication-factor 1 --partitions 1 --topic petrel_notification

# 查看所有 topics
docker exec -it kafka kafka-topics.sh --list --zookeeper 10.0.43.19:2181

# 查看 petrel_notification topic
docker exec -it kafka kafka-topics.sh --list --zookeeper 10.0.43.19:2181 --topic petrel_notification

# 列出所有 kafka brokers
docker exec -it zookeeper bin/zkCli.sh ls /brokers/ids

# 进入 kafka 容器
docker exec -it -e COLUMNS=200 -e LINES=200 kafka bash

# 发送消息，输入几条消息后，按^C退出发布
docker exec -it kafka kafka-console-producer.sh --broker-list 10.0.43.19:9092 --topic petrel_notification

# 接收消息
docker exec -it kafka kafka-console-consumer.sh --bootstrap-server 10.0.43.19:9092 --topic petrel_notification --from-beginning

# 删除 topics
# docker exec -it kafka kafka-topics.sh --delete --zookeeper 10.0.43.19:2181 --topic petrel_notification
```

[wurstmeister/zookeeper](https://hub.docker.com/r/wurstmeister/zookeeper)

[wurstmeister/kafka](https://hub.docker.com/r/wurstmeister/kafka)

### 实际操作

```sh
docker run -it --rm wurstmeister/kafka:2.11-1.1.1 bash

kafka-topics.sh --list --zookeeper 10.234.8.41:2181,10.234.8.42:2181,10.234.8.43:2181 | grep wms_e_stk

kafka-console-consumer.sh --bootstrap-server 10.234.8.41:9092,10.234.8.42:9092,10.234.8.43:9092 --topic wms_e_stk_syn_record --from-beginning

kafka-console-producer.sh --broker-list 10.234.8.41:9092,10.234.8.42:9092,10.234.8.43:9092 --topic wms_e_stk_syn_record

```

### 二进制安装

[Linux Kafka 2.11-1.1.1 安装搭建](https://blog.csdn.net/wangjianan7357/article/details/81214111?utm_source=blogxgwz6)

[nl2go/ansible-role-kafka](https://github.com/nl2go/ansible-role-kafka)

### Kafka-Eagle

[https://www.kafka-eagle.org/](https://www.kafka-eagle.org/)

[http://download.kafka-eagle.org/](http://download.kafka-eagle.org/)

[kafka-eagle-doc](https://www.kafka-eagle.org/articles/docs/documentation.html)

[smartloli/kafka-eagle](https://github.com/smartloli/kafka-eagle)

[Kafka监控系统Kafka Eagle剖析](https://www.cnblogs.com/smartloli/p/9371904.html)

[Kafka-Eagle 安装及其使用](https://www.codenong.com/cs107106341/)

[Kafka强大监控工具Kafka Eagle](https://blog.51cto.com/net881004/2538547?source=drt)

### kafka-manager

[yahoo/CMAK](https://github.com/yahoo/CMAK)

[hleb-albau/kafka-manager-docker](https://github.com/hleb-albau/kafka-manager-docker)

## 参考资料

[Kafka初识](https://www.cnblogs.com/luotianshuai/p/5206662.html#autoid-0-0-0)

[kafka 入门：简介、使用场景、设计原理、主要配置及集群搭建](https://www.cnblogs.com/likehua/p/3999538.html)

[为什么Kafka那么快](https://blog.csdn.net/z69183787/article/details/80323581)

[使用Docker快速搭建Kafka开发环境](https://tomoyadeng.github.io/blog/2018/06/02/kafka-cluster-in-docker/index.html)

[kafka原理及Docker环境部署](https://yq.aliyun.com/articles/657849)

[kafka主题](https://blog.csdn.net/qq_37502106/article/details/80351858)

[Kafka集群管理工具kafka-manager的安装使用](https://www.cnblogs.com/frankdeng/p/9584870.html)

[kaka-manager和kafka-offset-monitor的安装和使用](https://blog.csdn.net/hwz2311245/article/details/50983121)

[docker kafkaOffsetMonitor 安装与搭建监控](https://www.e-learn.cn/content/qita/675743)

[kafka监控工具KafkaOffsetMonitor配置及使用](https://www.cnblogs.com/dadonggg/p/8242682.html)

[Kafka启用SASL_PLAINTEXT动态配置JAAS文件的几种方式](https://blog.csdn.net/russle/article/details/81041135)

[Kafka SASL配置 & Demo测试](https://yanxml.blog.csdn.net/article/details/79562214?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control&dist_request_id=&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control)

[Kafka三种可视化监控管理工具Monitor/Manager/Eagle](https://cloud.tencent.com/developer/article/1667262)

[kafka监控方案](https://zhuanlan.zhihu.com/p/110022903)

[kafka配置文件server.properties中参数配置详解](https://blog.csdn.net/xrq1995/article/details/113250468)

[kafka配置参数详解](https://www.cnblogs.com/gxc2015/p/9835837.html)

[kafka部署集群所需要考虑的那些事儿](https://zhuanlan.zhihu.com/p/87585687)

[kafka高可用性集群](https://www.cnblogs.com/Leo_wl/p/11769396.html)

[Kafka动态配置实现原理解析](https://www.cnblogs.com/lizherui/p/12271285.html)
