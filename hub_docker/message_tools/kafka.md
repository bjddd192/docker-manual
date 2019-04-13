# kafka

[Kafka初识](https://www.cnblogs.com/luotianshuai/p/5206662.html#autoid-0-0-0)

## 官方镜像

[wurstmeister/zookeeper](https://hub.docker.com/r/wurstmeister/zookeeper)

[wurstmeister/kafka](https://hub.docker.com/r/wurstmeister/kafka)

[sheepkiller/kafka-manager](https://hub.docker.com/r/sheepkiller/kafka-manager)

## 启动命令

```sh
docker run -d --name zookeeper -p 2181:2181 -t hub.wonhigh.cn/library/zookeeper:3.4.12
 
docker run  -d --name kafka \
-p 9092:9092 \
-e KAFKA_BROKER_ID=0 \
-e KAFKA_ZOOKEEPER_CONNECT=10.0.43.24:2181 \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://10.0.43.24:9092 \
-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -t wurstmeister/kafka
 
docker run -d --name kafka-manager \
--link zookeeper:zookeeper \
--link kafka:kafka \
-p 9000:9000 \
-e ZK_HOSTS=zookeeper:2181 sheepkiller/kafka-manager:1.3.1.8

docker exec -it kafka kafka-topics.sh --create --zookeeper 10.240.114.50:2181 --replication-factor 1 --partitions 1 --topic topic_notification_uat
docker exec -it kafka kafka-topics.sh --create --zookeeper 10.240.114.50:2181 --replication-factor 1 --partitions 1 --topic topic_qrcode_uat
docker exec -it kafka kafka-topics.sh --create --zookeeper 10.240.114.50:2181 --replication-factor 1 --partitions 1 --topic topic_petrel_gateway_notification_uat
```

[wurstmeister/zookeeper](https://hub.docker.com/r/wurstmeister/zookeeper)

[wurstmeister/kafka](https://hub.docker.com/r/wurstmeister/kafka)

## 参考资料

[使用Docker快速搭建Kafka开发环境](https://tomoyadeng.github.io/blog/2018/06/02/kafka-cluster-in-docker/index.html)

[kafka主题](https://blog.csdn.net/qq_37502106/article/details/80351858)

[Kafka集群管理工具kafka-manager的安装使用](https://www.cnblogs.com/frankdeng/p/9584870.html)

[kaka-manager和kafka-offset-monitor的安装和使用](https://blog.csdn.net/hwz2311245/article/details/50983121)

[docker kafkaOffsetMonitor 安装与搭建监控](https://www.e-learn.cn/content/qita/675743)

[afka监控工具KafkaOffsetMonitor配置及使用](https://www.cnblogs.com/dadonggg/p/8242682.html)
