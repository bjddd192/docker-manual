# pulsar

Pulsar 是一个用于服务器到服务器的消息系统，具有多租户、高性能等优势。 Pulsar 最初由 Yahoo 开发，目前由 Apache 软件基金会管理。

Pulsar 的关键特性如下：

- Pulsar 的单个实例原生支持多个集群，可跨机房在集群间无缝地完成消息复制。
- 极低的发布延迟和端到端延迟。
- 可无缝扩展到超过一百万个 topic。
- 简单的客户端 API，支持 Java、Go、Python 和 C++。
- 支持多种 topic 订阅模式（独占订阅、共享订阅、故障转移订阅）。
- 通过 Apache BookKeeper 提供的持久化消息存储机制保证消息传递 。
- 由轻量级的 serverless 计算框架 Pulsar Functions 实现流原生的数据处理。
- 基于 Pulsar Functions 的 serverless connector 框架 Pulsar IO 使得数据更易移入、移出 Apache Pulsar。
- 分层式存储可在数据陈旧时，将数据从热存储卸载到冷/长期存储（如S3、GCS）中。

### 官方网站

[Apache Pulsar](http://pulsar.apache.org/zh-CN/)

[apachepulsar/pulsar hub_docker](https://hub.docker.com/r/apachepulsar/pulsar)

### kafka的缺陷

- Kafka 并不是为我们今天生活的云原生（Cloud Native）世界所设计的。
- Kafka 面临的主要挑战是它不善于处理大量主题。
- Kafka Broker 是绑定存储状态的，扩展或缩小 Kafka 集群需要重新平衡分区，这会影响我们的性能和请求时延，并限制我们对工作负载变化做出反应的方式和速度。

### pulsar部署

#### 单机部署

##### 二进制部署

##### docker部署

```sh
docker run -it --rm \
-p 6650:6650 \
-p 8080:8080 \
hub.wonhigh.cn/library/pulsar:2.8.0 \
bin/pulsar standalone
```

安装与消息验证可以参考：
[Set up a standalone Pulsar locally](http://pulsar.apache.org/docs/zh-CN/standalone/)
[Set up a standalone Pulsar in Docker](http://pulsar.apache.org/docs/zh-CN/standalone-docker/)
其中有关python的代码验证，可以进入容器后，运行 python 进入控制台，然后复制代码进行验证。

#### 集群部署

[Deploy a cluster on bare metal](http://pulsar.apache.org/docs/zh-CN/deploy-bare-metal/)

### Debezium Connector

[Managing Connectors](http://pulsar.apache.org/docs/zh-CN/io-managing/)

[Pulsar connector overview](http://pulsar.apache.org/docs/zh-CN/io-overview/#working-with-connectors)

[Source connector](http://pulsar.apache.org/docs/zh-CN/io-connectors/#source-connector)

[Sink connector](http://pulsar.apache.org/docs/zh-CN/io-connectors/#sink-connector)

[Debezium connector for MySQL](https://debezium.io/documentation/reference/1.6/connectors/mysql.html)

```sh
docker exec -it pulsar bash
# 本地运行模式中启动 Pulsar Debezium 连接器
bin/pulsar-admin source localrun \
--source-config-file connectors/debezium-mysql-source-config.yaml

# 订阅主题，查看消息
bin/pulsar-client consume -s "sub-products" public/default/dbserver1.inventory.products -n 0
```

### pulsar-manger

Pulsar Manger 是由 StreamNative 公司开源并捐献给 Apache 基金会的 Apache Pulsar 的管理端。它是基于 Web 的 GUI 管理工具，支持多种环境的动态配置，主要面向的用户群体是 Pulsar 的管理员，用于管理和监控 Pulsar。通过 Pulsar Manager， 可管理 tenants、namespaces、topics、subscriptions、brokers、clusters 等。

一个 Pulsar Manager 能够管理多个 Pulsar 集群。在 Pulsar Manager 中，将一个 Pulsar 实例或一组 Pulsar 群集定义为 Environment 。您可以创建尽可能多的 Environment 。

```sh
docker run -dit \
-p 9527:9527 -p 7750:7750 \
-e SPRING_CONFIGURATION_FILE=/pulsar-manager/pulsar-manager/application.properties \
apachepulsar/pulsar-manager:v0.2.0

# 生成秘钥
CSRF_TOKEN=$(curl http://localhost:7750/pulsar-manager/csrf-token)

# 初始化账号
curl \
   -H 'X-XSRF-TOKEN: $CSRF_TOKEN' \
   -H 'Cookie: XSRF-TOKEN=$CSRF_TOKEN;' \
   -H "Content-Type: application/json" \
   -X PUT http://localhost:7750/pulsar-manager/users/superuser \
   -d '{"name": "admin", "password": "apachepulsar", "description": "test", "email": "yang.lei@belle.com.cn"}'

# web验证
http://10.0.30.205:9527/
# bkvm，默认是关闭的，需自行开启
http://10.0.30.205:7750/

# cluster页面无法访问的处理
bin/pulsar-admin clusters get standalone
bin/pulsar-admin clusters update standalone --url http://10.0.30.205:8080 --broker-url pulsar://127.0.0.1:6650
```

### 参考资料

[选择Pulsar而不是Kafka的7大理由](https://time.geekbang.org/column/article/98245)

[为什么已有Kafka，我们最终却选择了Apache Pulsar？](https://www.jianshu.com/p/8e25520372f5)

[Pulsar的存储计算分离设计：全新的消息队列设计思路](https://time.geekbang.org/column/article/140913)

[如何在 Pulsar 中使用 Debezium Connector](https://blog.csdn.net/zhaijia03/article/details/109766331)

[Pulsar IO之CDC Debezium Connector](https://blog.csdn.net/qq_32470693/article/details/96001454)

[Pulsar Manager 介绍及实操应用](https://blog.csdn.net/zhaijia03/article/details/109766643)

[Pulsar 管理工具的介绍和使用](https://v.qq.com/x/page/o3161e7dnhd.html?start=2135)

[Pulsar-Manager安装部署](https://blog.51cto.com/u_536410/2561864)
