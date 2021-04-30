# debezium

Debezium是一个分布式平台，它将您现有的数据库转换为事件流，因此应用程序可以看到数据库中的每一个行级更改并立即做出响应。Debezium构建在Apache Kafka之上，并提供Kafka连接兼容的连接器来监视特定的数据库管理系统。Debezium在Kafka日志中记录数据更改的历史，您的应用程序将从这里使用它们。这使您的应用程序能够轻松、正确、完整地使用所有事件。即使您的应用程序停止(或崩溃)，在重新启动时，它将开始消耗它停止的事件，因此它不会错过任何东西。

[Debezium 官网](https://debezium.io/)

[Debezium Releases Overview](https://debezium.io/releases/)

[Debezium Documentation](https://debezium.io/documentation/reference/1.5/index.html)

[debezium 官方镜像](https://hub.docker.com/u/debezium)

### 部署说明

#### 部署示例仓库

[debezium/debezium-examples Github](https://github.com/debezium/debezium-examples/tree/master/tutorial)

#### MySQL部署测试

```sh
# Starting Zookeeper
docker run -it --rm --name zookeeper -p 2181:2181 -p 2888:2888 -p 3888:3888 debezium/zookeeper:1.5
# Starting Kafka
docker run -it --rm --name kafka -p 9092:9092 --link zookeeper:zookeeper debezium/kafka:1.5
# Starting a MySQL database
docker run -it --rm --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=debezium -e MYSQL_USER=mysqluser -e MYSQL_PASSWORD=mysqlpw debezium/example-mysql:1.5
# Starting a MySQL command line client
docker run -it --rm --name mysqlterm --link mysql --rm mysql:5.7 sh -c 'exec mysql -h"$MYSQL_PORT_3306_TCP_ADDR" -P"$MYSQL_PORT_3306_TCP_PORT" -uroot -p"$MYSQL_ENV_MYSQL_ROOT_PASSWORD"'
mysql> use inventory;
mysql> show tables;
+---------------------+
| Tables_in_inventory |
+---------------------+
| addresses           |
| customers           |
| geom                |
| orders              |
| products            |
| products_on_hand    |
+---------------------+
mysql> SELECT * FROM customers;
+------+------------+-----------+-----------------------+
| id   | first_name | last_name | email                 |
+------+------------+-----------+-----------------------+
| 1001 | Sally      | Thomas    | sally.thomas@acme.com |
| 1002 | George     | Bailey    | gbailey@foobar.com    |
| 1003 | Edward     | Walker    | ed@walker.com         |
| 1004 | Anne       | Kretchmar | annek@noanswer.org    |
+------+------------+-----------+-----------------------+
mysql> show grants for 'debezium'@'%';
+------------------------------------------------------------------------------------------------------+
| Grants for debezium@%                                                                                |
+------------------------------------------------------------------------------------------------------+
| GRANT SELECT, RELOAD, SHOW DATABASES, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'debezium'@'%' |
+------------------------------------------------------------------------------------------------------+
mysql> show variables like 'server_id';
+---------------+--------+
| Variable_name | Value  |
+---------------+--------+
| server_id     | 223344 |
+---------------+--------+
# Starting Kafka Connect
docker run -it --rm --name connect -p 8083:8083 -e GROUP_ID=1 -e CONFIG_STORAGE_TOPIC=my_connect_configs -e OFFSET_STORAGE_TOPIC=my_connect_offsets -e STATUS_STORAGE_TOPIC=my_connect_statuses --link zookeeper:zookeeper --link kafka:kafka --link mysql:mysql debezium/connect:1.5
# Open a new terminal and check the status of the Kafka Connect service
curl -H "Accept:application/json" localhost:8083/
{"version":"2.7.0","commit":"448719dc99a19793","kafka_cluster_id":"uc6guENHQV6lnMxI9fwsFQ"}
# Check the list of connectors registered with Kafka Connect (No connectors are currently registered with Kafka Connect)
curl -H "Accept:application/json" localhost:8083/connectors/
[]
# Review the configuration of the Debezium MySQL connector that you will register.
# 使用单个连接器任务可以确保正确的顺序和事件处理。
{
  "name": "inventory-connector",  
  "config": {  
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
    "tasks.max": "1",  
    "database.hostname": "mysql",  
    "database.port": "3306",
    "database.user": "debezium",
    "database.password": "dbz",
    "database.server.id": "184054",  
    "database.server.name": "dbserver1",  
    "database.include.list": "inventory",  
    "database.history.kafka.bootstrap.servers": "kafka:9092",  
    "database.history.kafka.topic": "schema-changes.inventory"  
  }
}
# Deploying the MySQL connector
# Open a new terminal, and use the curl command to register the Debezium MySQL connector.
curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" localhost:8083/connectors/ -d '{ "name": "inventory-connector", "config": { "connector.class": "io.debezium.connector.mysql.MySqlConnector", "tasks.max": "1", "database.hostname": "mysql", "database.port": "3306", "database.user": "debezium", "database.password": "dbz", "database.server.id": "184054", "database.server.name": "dbserver1", "database.include.list": "inventory", "database.history.kafka.bootstrap.servers": "kafka:9092", "database.history.kafka.topic": "dbhistory.inventory" } }'
# Review the connector’s tasks
curl -i -X GET -H "Accept:application/json" localhost:8083/connectors/inventory-connector
# Open a new terminal, and use it to start the watch-topic utility to watch the dbserver1.inventory.customers topic from the beginning of the topic.
docker run -it --rm --name watcher --link zookeeper:zookeeper --link kafka:kafka debezium/kafka:1.5 watch-topic -a -k dbserver1.inventory.customers
# -a 关注主题创建后的所有事件。如果没有这个选项，watch-topic 将只显示您开始观看之后记录的事件。
# -k 指定输出应该包含事件的键。在这种情况下，它包含行的主键。

# 事件触发(查看捕获效果)
# 更新数据
mysql> UPDATE customers SET first_name='Anne Marie' WHERE id=1004;
# 删除数据
mysql> DELETE FROM addresses WHERE customer_id=1004;
mysql> DELETE FROM customers WHERE id=1004;

docker stop connect

mysql> INSERT INTO customers VALUES (default, "Sarah", "Thompson", "kitt@acme.com");
mysql> INSERT INTO customers VALUES (default, "Kenneth", "Anderson", "kander@acme.com");

docker run -it --rm --name connect -p 8083:8083 -e GROUP_ID=1 -e CONFIG_STORAGE_TOPIC=my_connect_configs -e OFFSET_STORAGE_TOPIC=my_connect_offsets -e STATUS_STORAGE_TOPIC=my_connect_statuses --link zookeeper:zookeeper --link kafka:kafka --link mysql:mysql debezium/connect:1.5

# 运行可视化界面(debezium-ui 还不成熟，启动后无法使用，后续有版本再看)
docker run -it --rm --name ui -p 8080:8080 --link connect:connect debezium/debezium-ui:1.5

# Cleaning up
docker stop mysqlterm watcher connect mysql kafka zookeeper ui
```

#### MongoDB部署测试

[debezium mongodb 集成测试](https://www.cnblogs.com/rongfengliang/p/10176261.html)

[《Debezium系列》Debezium-Connector-MongoDB副本集](https://blog.csdn.net/weixin_41461992/article/details/103973596)

[kafka-connect-distribute 模式，使用 debezium source 同步 MongoDB 集群到 Kafka](https://blog.csdn.net/bao_since/article/details/90055158)

debezium mongodb 的 cdc 是基于复制集实现的，实际上 mongodb 已经支持了 stream 可以进行数据的捕获处理。

### kafka-connect REST API

```sh
GET /connectors – 返回所有正在运行的connector名

POST /connectors – 新建一个connector; 请求体必须是json格式并且需要包含name字段和config字段，name是connector的名字，config是json格式，必须包含你的connector的配置信息。

GET /connectors/{name} – 获取指定connetor的信息

GET /connectors/{name}/config – 获取指定connector的配置信息

PUT /connectors/{name}/config – 更新指定connector的配置信息

GET /connectors/{name}/status – 获取指定connector的状态，包括它是否在运行、停止、或者失败，如果发生错误，还会列出错误的具体信息。

GET /connectors/{name}/tasks – 获取指定connector正在运行的task。

GET /connectors/{name}/tasks/{taskid}/status – 获取指定connector的task的状态信息

PUT /connectors/{name}/pause – 暂停connector和它的task，停止数据处理知道它被恢复。

PUT /connectors/{name}/resume – 恢复一个被暂停的connector

POST /connectors/{name}/restart – 重启一个connector，尤其是在一个connector运行失败的情况下比较常用

POST /connectors/{name}/tasks/{taskId}/restart – 重启一个task，一般是因为它运行失败才这样做。

DELETE /connectors/{name} – 删除一个connector，停止它的所有task并删除配置。
 
POST /connectors – 新建一个connector; 请求体必须是json格式并且需要包含name字段和config字段，name是connector的名字，config是json格式，必须包含你的connector的配置信息。
 GET /connectors/{name} – 获取指定connetor的信息
 GET /connectors/{name}/config – 获取指定connector的配置信息
 PUT /connectors/{name}/config – 更新指定connector的配置信息
 GET /connectors/{name}/status – 获取指定connector的状态，包括它是否在运行、停止、或者失败，如果发生错误，还会列出错误的具体信息。
 GET /connectors/{name}/tasks – 获取指定connector正在运行的task。
 GET /connectors/{name}/tasks/{taskid}/status – 获取指定connector的task的状态信息
 PUT /connectors/{name}/pause – 暂停connector和它的task，停止数据处理知道它被恢复。
 PUT /connectors/{name}/resume – 恢复一个被暂停的connector
 POST /connectors/{name}/restart – 重启一个connector，尤其是在一个connector运行失败的情况下比较常用
 POST /connectors/{name}/tasks/{taskId}/restart – 重启一个task，一般是因为它运行失败才这样做。
 DELETE /connectors/{name} – 删除一个connector，停止它的所有task并删除配置。 
```

### 参考资料

[「技术架构」CDC (捕获数据变化) Debezium 介绍](http://jiagoushi.pro/book/export/html/776)

[构建生产级具有融合Kafka的Debezium集群](https://www.modb.pro/db/13312)

[Debezium系列学习](https://blog.csdn.net/sweatott/category_7505316.html)

[干货 | Debezium实现Mysql到Elasticsearch高效实时同步](https://blog.csdn.net/laoyang360/article/details/87897886)

[基于 Kafka 与 Debezium 构建实时数据管道](https://aleiwu.com/post/vimur.cn/)

[Debezium的MySQL连接器的工作原理](https://blog.csdn.net/lzufeng/article/details/81606825)

[Apache Avro & Avro Schema简介](https://www.cnblogs.com/fangjb/p/13305433.html)

[debezium 从指定 binlog position 同步事件消息](https://www.it610.com/article/1277696681881649152.htm)

[Streaming ETL pipeline](https://docs.ksqldb.io/en/latest/tutorials/etl/)
