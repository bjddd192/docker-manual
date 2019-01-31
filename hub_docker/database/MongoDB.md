# MongoDB

## 官方镜像

[Hub官方](https://hub.docker.com/_/mongo/)

## 启动命令

### 单节点

```shell
docker stop mongo && docker rm mongo

docker run -d --name mongo -p 30017:27017 --restart=always \
  -e MONGO_INITDB_ROOT_USERNAME=root \
  -e MONGO_INITDB_ROOT_PASSWORD=123456 \
  -v /home/docker/mongo/data:/data/db \
  mongo:3.4.18
```

测试数据：

```shell
docker exec -it mongo mongo admin --port 27017 -u root -p123456
> use test
> db.test.insert({msg: "this message is from master", ts: new Date()})
> show dbs
```

### 一主一从

**优缺点：master-slave 结构，当 master 挂了，slave 不会被选举为 master，所以这种结构只起到了备份数据的作用。**

```shell
docker stop mongo_master && docker rm mongo_master

docker run -d --name mongo_master --net=host --restart=always \
  -v /home/docker/mongo/master/data:/data/db \
  mongo:3.4.18 \
  mongod --port 30018 --master
  
docker stop mongo_slaver && docker rm mongo_slaver
  
docker run -d --name mongo_slaver --net=host --restart=always \
  -v /home/docker/mongo/slaver/data:/data/db \
  mongo:3.4.18 \
  mongod --port 30019 --slave --source 10.68.0.7:30018  
```

测试数据：

```shell
docker exec -it mongo_master mongo --port 30018
> use test
> db.test.insert({msg: "this message is from master", ts: new Date()})
> db.test.find();
> show dbs
> exit

docker exec -it mongo_slaver mongo --port 30019
> use test
switched to db test
> db.test.insert({msg: "this message is from slaver", ts: new Date()})
# slaver 只读，无法写入
WriteResult({ "writeError" : { "code" : 10107, "errmsg" : "not master" } })
# 副本节点上不允许读，需要设置副本节点可以读。
> db.test.setSlaveOk();
> db.test.find();
# 查看服务状态
> db.printReplicationInfo();
> exit
```

mongoDB 官方已经**不建议使用主从模式**了，替代方案是采用副本集的模式。

### 副本集

#### 无认证

```shell
mkdir -p /home/docker/mongo/replica_set01
mkdir -p /home/docker/mongo/replica_set02
mkdir -p /home/docker/mongo/replica_set03

docker stop mongo_replica_set01 && docker rm mongo_replica_set01

docker run -d --name mongo_replica_set01 --net=host --restart=always \
  -v /home/docker/mongo/replica_set01/data:/data/db \
  mongo:3.4.18 \
  mongod --bind_ip 10.68.0.7 --port 30020 --replSet mongoreplset

docker stop mongo_replica_set02 && docker rm mongo_replica_set02

docker run -d --name mongo_replica_set02 --net=host --restart=always \
  -v /home/docker/mongo/replica_set02/data:/data/db \
  mongo:3.4.18 \
  mongod --bind_ip 10.68.0.7 --port 30021 --replSet mongoreplset

docker stop mongo_replica_set03 && docker rm mongo_replica_set03

docker run -d --name mongo_replica_set03 --net=host --restart=always \
  -v /home/docker/mongo/replica_set03/data:/data/db \
  mongo:3.4.18 \
  mongod --bind_ip 10.68.0.7 --port 30022 --replSet mongoreplset
```

初始化 replica set：

```shell
docker exec -it mongo_replica_set01 mongo --port 30020 --host 10.68.0.7
> rs.initiate()
> rs.add('10.68.0.7:30021')
> rs.add('10.68.0.7:30022')
> rs.status()
> use test
> db.test.insert({msg: "this message is from replica_set01", ts: new Date()})
> db.test.find();
> exit

docker exec -it mongo_replica_set02 mongo --port 30021 --host 10.68.0.7
> db.getMongo().setSlaveOk();
> db.test.find();
> exit

docker exec -it mongo_replica_set03 mongo --port 30022 --host 10.68.0.7
> db.getMongo().setSlaveOk();
> db.test.find();
> exit
```

设置账号：

```shell
docker exec -it mongo_replica_set01 mongo --port 30020 --host 10.68.0.7
> use admin
> db.createUser({user:"root",pwd:"root",roles:[{role:"root",db:"admin"}]})
> db.createUser({user:"admin",pwd:"admin",roles:[{role:"userAdminAnyDatabase",db:"admin"}]})
> use test
> db.createUser({user:"test",pwd:"test",roles:[{role:"readWrite",db:"test"}]})
> db.auth("test","test")
```

#### 有认证

Replica Set要使用keyFile的校验方式，让集群的member之间同步，也就是说，通过keyFile获得__system用户在local上的权限。local存放着Replica Set的配置和同步信息。

MongoDB官方推荐的keyFile的生产方式：

```shell
openssl rand -base64 756 > /data/key_file
chmod 400 /data/key_file
```

```shell
mkdir -p /home/docker/mongo/replica_set01/data
mkdir -p /home/docker/mongo/replica_set02/data
mkdir -p /home/docker/mongo/replica_set03/data

# key_file 必须跟数据文件在一个目录，否则容器启动报错，如：[main] error opening file: /data/key_file: Permission denied
\cp -f /data/key_file /home/docker/mongo/replica_set01/data/
\cp -f /data/key_file /home/docker/mongo/replica_set02/data/
\cp -f /data/key_file /home/docker/mongo/replica_set03/data/

docker stop mongo_replica_set01 && docker rm mongo_replica_set01

docker run -d --name mongo_replica_set01 --net=host --restart=always \
  -e MONGO_INITDB_ROOT_USERNAME=root \
  -e MONGO_INITDB_ROOT_PASSWORD=123456 \
  -v /home/docker/mongo/replica_set01/data:/data/db \
  mongo:3.4.18 \
  mongod --bind_ip 10.68.0.7 --port 30020 --replSet mongoreplset --auth --keyFile=/data/db/key_file

docker stop mongo_replica_set02 && docker rm mongo_replica_set02

docker run -d --name mongo_replica_set02 --net=host --restart=always \
  -e MONGO_INITDB_ROOT_USERNAME=root \
  -e MONGO_INITDB_ROOT_PASSWORD=123456 \
  -v /home/docker/mongo/replica_set02/data:/data/db \
  mongo:3.4.18 \
  mongod --bind_ip 10.68.0.7 --port 30021 --replSet mongoreplset --auth --keyFile=/data/db/key_file

docker stop mongo_replica_set03 && docker rm mongo_replica_set03

docker run -d --name mongo_replica_set03 --net=host --restart=always \
  -e MONGO_INITDB_ROOT_USERNAME=root \
  -e MONGO_INITDB_ROOT_PASSWORD=123456 \
  -v /home/docker/mongo/replica_set03/data:/data/db \
  mongo:3.4.18 \
  mongod --bind_ip 10.68.0.7 --port 30022 --replSet mongoreplset --auth --keyFile=/data/db/key_file
```

初始化 replica set：

```shell
docker exec -it mongo_replica_set01 mongo admin --port 30020 --host 10.68.0.7 -u root -p 123456
> rs.initiate()
> rs.add('10.68.0.7:30021')
> rs.add('10.68.0.7:30022')
> rs.status()
> use test
> db.test.insert({msg: "this message is from replica_set01", ts: new Date()})
> db.test.find();
> exit

docker exec -it mongo_replica_set02 mongo --port 30021 --host 10.68.0.7
> db.getMongo().setSlaveOk();
> db.test.find();
> exit

docker exec -it mongo_replica_set03 mongo --port 30022 --host 10.68.0.7
> db.getMongo().setSlaveOk();
> db.test.find();
> exit
```

### 分片集群

后续继续深入研究。

## 参考资料

[MongoDB的Replica Set以及Auth的配置](https://www.cnblogs.com/pbblogs/p/9783964.html)

[Enforce Keyfile Access Control in a Replica Set](https://docs.mongodb.com/manual/tutorial/enforce-keyfile-access-control-in-existing-replica-set/)

[Enforce Keyfile Access Control in a Replica Set without Downtime](https://docs.mongodb.com/manual/tutorial/enforce-keyfile-access-control-in-existing-replica-set-without-downtime/)

