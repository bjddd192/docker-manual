# canal

canal [kə'næl]，译意为水道/管道/沟渠，主要用途是基于 MySQL 数据库增量日志解析，提供增量数据订阅和消费

早期阿里巴巴因为杭州和美国双机房部署，存在跨机房同步的业务需求，实现方式主要是基于业务 trigger 获取增量变更。从 2010 年开始，业务逐步尝试数据库日志解析获取增量变更进行同步，由此衍生出了大量的数据库增量订阅和消费业务。

[alibaba/canal](https://github.com/alibaba/canal)

[client-adapter](https://github.com/alibaba/canal/tree/master/client-adapter)

### 部署说明

1、创建db_canal_manager数据库
2、创建zookeeper集群
3、创建kafka集群
4、启动canal-admin；配置“集群管理”；设置”主配置”
5、启动canal-server，会自动注册上来；
6、配置instance，并启动；
7、监听主题，修改数据，看是否触发消息写入;

### 创建DB

脚本地址：[canal_manager.sql](https://github.com/alibaba/canal/blob/canal-1.1.4/canal-admin/canal-admin-server/src/main/resources/canal_manager.sql)

细微调整：

```sql
CREATE DATABASE /*!32312 IF NOT EXISTS*/ `db_canal_manager` /*!40100 DEFAULT CHARACTER SET utf8 COLLATE utf8_bin */;
CREATE DATABASE /*!32312 IF NOT EXISTS*/ `db_canal_tsdb` /*!40100 DEFAULT CHARACTER SET utf8 COLLATE utf8_bin */;

USE `db_canal_manager`;

SET NAMES utf8;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
-- Table structure for canal_adapter_config
-- ----------------------------
DROP TABLE IF EXISTS `canal_adapter_config`;
CREATE TABLE `canal_adapter_config` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `category` varchar(45) NOT NULL,
  `name` varchar(45) NOT NULL,
  `status` varchar(45) DEFAULT NULL,
  `content` text NOT NULL,
  `modified_time` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Table structure for canal_cluster
-- ----------------------------
DROP TABLE IF EXISTS `canal_cluster`;
CREATE TABLE `canal_cluster` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `name` varchar(63) NOT NULL,
  `zk_hosts` varchar(255) NOT NULL,
  `modified_time` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Table structure for canal_config
-- ----------------------------
DROP TABLE IF EXISTS `canal_config`;
CREATE TABLE `canal_config` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `cluster_id` bigint(20) DEFAULT NULL,
  `server_id` bigint(20) DEFAULT NULL,
  `name` varchar(45) NOT NULL,
  `status` varchar(45) DEFAULT NULL,
  `content` text NOT NULL,
  `content_md5` varchar(128) NOT NULL,
  `modified_time` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `sid_UNIQUE` (`server_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Table structure for canal_instance_config
-- ----------------------------
DROP TABLE IF EXISTS `canal_instance_config`;
CREATE TABLE `canal_instance_config` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `cluster_id` bigint(20) DEFAULT NULL,
  `server_id` bigint(20) DEFAULT NULL,
  `name` varchar(45) NOT NULL,
  `status` varchar(45) DEFAULT NULL,
  `content` text NOT NULL,
  `content_md5` varchar(128) DEFAULT NULL,
  `modified_time` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `name_UNIQUE` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Table structure for canal_node_server
-- ----------------------------
DROP TABLE IF EXISTS `canal_node_server`;
CREATE TABLE `canal_node_server` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `cluster_id` bigint(20) DEFAULT NULL,
  `name` varchar(63) NOT NULL,
  `ip` varchar(63) NOT NULL,
  `admin_port` int(11) DEFAULT NULL,
  `tcp_port` int(11) DEFAULT NULL,
  `metric_port` int(11) DEFAULT NULL,
  `status` varchar(45) DEFAULT NULL,
  `modified_time` datetime DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Table structure for canal_user
-- ----------------------------
DROP TABLE IF EXISTS `canal_user`;
CREATE TABLE `canal_user` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `username` varchar(31) NOT NULL,
  `password` varchar(128) NOT NULL,
  `name` varchar(31) NOT NULL,
  `roles` varchar(31) NOT NULL,
  `introduction` varchar(255) DEFAULT NULL,
  `avatar` varchar(255) DEFAULT NULL,
  `creation_date` timestamp  DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

SET FOREIGN_KEY_CHECKS = 1;

-- ----------------------------
-- Records of canal_user
-- ----------------------------
BEGIN;
INSERT INTO `canal_user` VALUES (1, 'admin', '6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9', 'Canal Manager', 'admin', NULL, NULL, '2019-07-14 00:05:28');
COMMIT;

SET FOREIGN_KEY_CHECKS = 1;
```

### 容器化

[canal/canal-admin](https://hub.docker.com/r/canal/canal-admin)

[canal/canal-server](https://hub.docker.com/r/canal/canal-server)

#### canal-admin

```sh
docker pull canal/canal-admin:v1.1.4
  
docker stop canal-admin && docker rm canal-admin

docker run -d --name canal-admin --restart=always \
-e server.port=8089 \
-e canal.adminUser=admin \
-e canal.adminPasswd=admin \
-e spring.datasource.address=10.0.30.39:3306 \
-e spring.datasource.database=db_canal_manager \
-e spring.datasource.username=user_canal \
-e spring.datasource.password=scm_canal \
--net=host \
-h 172.17.209.202 \
-m 2048m \
canal/canal-admin:v1.1.4

# 持久化数据
docker cp canal-admin:/home/admin/canal-admin /data/docker_volumn/

docker stop canal-admin && docker rm canal-admin

docker run -d --name canal-admin --restart=always \
-e server.port=8089 \
-e canal.adminUser=admin \
-e canal.adminPasswd=admin \
-e spring.datasource.address=10.0.30.39:3306 \
-e spring.datasource.database=db_canal_manager \
-e spring.datasource.username=user_canal \
-e spring.datasource.password=scm_canal \
--net=host \
-h 172.17.209.202 \
-m 2048m \
-v /data/docker_volumn/canal-admin:/home/admin/canal-admin \
canal/canal-admin:v1.1.4

# 容器日志目录： /home/admin/canal-admin/logs

# 验证：
http://172.17.209.202:8089/
admin/123456
```

#### canal-server

```sh
docker pull canal/canal-server:v1.1.4

docker stop canal-server && docker rm canal-server

docker run -d --name canal-server -p 11110:11110 -p 11111:11111 -p 11112:11112 -p 9300:9100 --restart=always \
-e canal.admin.manager=172.17.209.202:8089 \
-e canal.admin.user=admin \
-e canal.admin.passwd=4ACFE3202A5FF5CF467898FC58AAB1D615029441 \
-e canal.admin.port=11110 \
-e canal.admin.register.cluster=dev_canal_zk_cluster \
-e canal.register.ip=172.17.209.202 \
-m 4096m \
canal/canal-server:v1.1.4

docker cp canal-server:/home/admin/canal-server /data/docker_volumn/

docker stop canal-server && docker rm canal-server

docker run -d --name canal-server -p 11110:11110 -p 11111:11111 -p 11112:11112 -p 9300:9100 --restart=always \
-e canal.admin.manager=172.17.209.202:8089 \
-e canal.admin.user=admin \
-e canal.admin.passwd=4ACFE3202A5FF5CF467898FC58AAB1D615029441 \
-e canal.admin.port=11110 \
-e canal.admin.register.cluster=dev_canal_zk_cluster \
-e canal.register.ip=172.17.209.202 \
-m 4096m \
-v /data/docker_volumn/canal-server:/home/admin/canal-server \
canal/canal-server:v1.1.4

# docker stop canal-server && docker rm canal-server
# 
# docker run -d --name canal-server --restart=always \
# -e canal.auto.scan=false \
# -e canal.destinations=test \
# -e canal.instance.master.address=10.0.30.39:3306 \
# -e canal.instance.dbUsername=user_canal \
# -e canal.instance.dbPassword=scm_canal \
# -e canal.instance.connectionCharset=UTF-8 \
# -e canal.instance.tsdb.enable=true \
# -e canal.instance.gtidon=false \
# -e canal.mq.topic=example
# -e canal.mq.partition=0
# -e canal.serverMode=kafka
# -e canal.mq.servers=kafka:9092
# -e canal.mq.flatMessage=true
# --net=host \
# -h 172.17.209.202 \
# -m 4096m \
# -v /data/docker_volumn/canal-server:/home/admin/canal-server \
# canal/canal-server:v1.1.4

### kafka 操作
docker exec -it -e LINES=200 -e COLUMNS=200 kafka bash
cd /opt/kafka_2.11-1.1.1/bin/
# 查看所有topic
/opt/kafka_2.11-1.1.1/bin/kafka-topics.sh --list --zookeeper 10.234.6.220:12181
# 接收指定主题消息
/opt/kafka_2.11-1.1.1/bin/kafka-console-consumer.sh --bootstrap-server 10.234.6.220:9092 --from-beginning --topic ldp_canal_wms_to_ck
```

### 异常处理

[canal1.1.4出现报错column size is not match for table](https://www.askcug.com/topic/28/canal1-1-4%E5%87%BA%E7%8E%B0%E6%8A%A5%E9%94%99column-size-is-not-match-for-table?_=1617248655258)

### 参考资料

[AdminGuide](https://github.com/alibaba/canal/wiki/AdminGuide)

[Canal Admin Guide](https://github.com/alibaba/canal/wiki/Canal-Admin-Guide)

[Canal Admin QuickStart](https://github.com/alibaba/canal/wiki/Canal-Admin-QuickStart)

[Canal Server+Canal Admin](https://www.cnblogs.com/dalianpai/p/13620035.html)

[Canal Kafka RocketMQ QuickStart](https://github.com/alibaba/canal/wiki/Canal-Kafka-RocketMQ-QuickStart)

[docker安装canal同步mysql8与elasticsearch7数据](https://blog.csdn.net/daziyuanazhen/article/details/106098887)

[利用Canal投递MySQL Binlog到Kafka](https://www.jianshu.com/p/93d9018e2fa1)

[使用 Docker 部署 canal 服务，实现 MySQL 数据库 binlog 日志解析](https://xie.infoq.cn/article/e3fc7fe27d0fe622d8a5fb22a)

[使用 Docker 部署 canal，并将消息推送到 RabbitMQ](https://xie.infoq.cn/article/e30edd8378082f8c0e140ef77)

[一文带你快速入门 Canal，看这篇就够了！](https://xie.infoq.cn/article/1cfca1549886dddda2cb5b60e)

[详细讲解！Canal+Kafka实现MySQL与Redis数据同步！](https://developer.aliyun.com/article/770659)

[younthu/canal_demo](https://github.com/younthu/canal_demo)

[高可用Canal集群部署实战](https://mp.weixin.qq.com/s/pI48-CU24xruBLT8Cdla4Q)

[Canal v1.1.4版本避坑指南](https://www.cnblogs.com/throwable/p/13449920.html)
