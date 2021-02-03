

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

### 分片集群(单机)

```sh
# 部署集群 shard1
mkdir -p /data/docker_volumn/mongo_shard_cluster/shardsvr11/data
openssl rand -base64 756 > /data/docker_volumn/mongo_shard_cluster/shardsvr11/data/key_file
chmod 400 /data/docker_volumn/mongo_shard_cluster/shardsvr11/data/key_file
ll /data/docker_volumn/mongo_shard_cluster/shardsvr11/data/key_file
cat /data/docker_volumn/mongo_shard_cluster/shardsvr11/data/key_file

mkdir -p /data/docker_volumn/mongo_shard_cluster/shardsvr12/data
mkdir -p /data/docker_volumn/mongo_shard_cluster/shardsvr13/data
\cp -f /data/docker_volumn/mongo_shard_cluster/shardsvr11/data/key_file /data/docker_volumn/mongo_shard_cluster/shardsvr12/data/
\cp -f /data/docker_volumn/mongo_shard_cluster/shardsvr11/data/key_file /data/docker_volumn/mongo_shard_cluster/shardsvr13/data/
ll /data/docker_volumn/mongo_shard_cluster/shardsvr12/data/key_file
ll /data/docker_volumn/mongo_shard_cluster/shardsvr13/data/key_file

# docker stop mongo_shardsvr11 && docker rm mongo_shardsvr11
docker run -d --name mongo_shardsvr11 --net=host --privileged --restart=always \
-e MONGO_INITDB_ROOT_USERNAME=root \
-e MONGO_INITDB_ROOT_PASSWORD=SYiXHC0hWE \
-v /data/docker_volumn/mongo_shard_cluster/shardsvr11/data:/data/db \
mongo:4.4.3 \
numactl --interleave=all mongod --bind_ip 10.234.6.33 --port 27017 --replSet shard1 --shardsvr --auth --keyFile=/data/db/key_file

# docker stop mongo_shardsvr12 && docker rm mongo_shardsvr12
docker run -d --name mongo_shardsvr12 --net=host --privileged --restart=always \
-e MONGO_INITDB_ROOT_USERNAME=root \
-e MONGO_INITDB_ROOT_PASSWORD=SYiXHC0hWE \
-v /data/docker_volumn/mongo_shard_cluster/shardsvr12/data:/data/db \
mongo:4.4.3 \
numactl --interleave=all mongod --bind_ip 10.234.6.33 --port 27027 --replSet shard1 --shardsvr --auth --keyFile=/data/db/key_file

# docker stop mongo_shardsvr13 && docker rm mongo_shardsvr13
docker run -d --name mongo_shardsvr13 --net=host --privileged --restart=always \
-e MONGO_INITDB_ROOT_USERNAME=root \
-e MONGO_INITDB_ROOT_PASSWORD=SYiXHC0hWE \
-v /data/docker_volumn/mongo_shard_cluster/shardsvr13/data:/data/db \
mongo:4.4.3 \
numactl --interleave=all mongod --bind_ip 10.234.6.33 --port 27037 --replSet shard1 --shardsvr --auth --keyFile=/data/db/key_file

# 初始化集群 shard1
docker exec -it -e COLUMNS=200 -e LINES=200 mongo_shardsvr11 mongo admin --port 27017 --host 10.234.6.33 -u root -p SYiXHC0hWE
> rs.initiate(
   {
      _id: "shard1",
      members: [
         { _id: 0, host : "10.234.6.33:27017" },
         { _id: 1, host : "10.234.6.33:27027" },
         { _id: 2, host : "10.234.6.33:27037" }
      ]
   }
);
shard1:PRIMARY> rs.status();

# 部署集群 configsvr
# 初始化时不能开启认证，--configsvr 时 MONGO_INITDB_ROOT_USERNAME 失效了
# 需要初始化完集群后再用命令创建 root 用户
mkdir -p /data/docker_volumn/mongo_shard_cluster/configsvr1/data
mkdir -p /data/docker_volumn/mongo_shard_cluster/configsvr2/data
mkdir -p /data/docker_volumn/mongo_shard_cluster/configsvr3/data
\cp -f /data/docker_volumn/mongo_shard_cluster/shardsvr11/data/key_file /data/docker_volumn/mongo_shard_cluster/configsvr1/data
\cp -f /data/docker_volumn/mongo_shard_cluster/shardsvr11/data/key_file /data/docker_volumn/mongo_shard_cluster/configsvr2/data
\cp -f /data/docker_volumn/mongo_shard_cluster/shardsvr11/data/key_file /data/docker_volumn/mongo_shard_cluster/configsvr3/data
ll /data/docker_volumn/mongo_shard_cluster/configsvr1/data
ll /data/docker_volumn/mongo_shard_cluster/configsvr2/data
ll /data/docker_volumn/mongo_shard_cluster/configsvr3/data

# docker stop mongo_configsvr1 && docker rm mongo_configsvr1
docker run -d --name mongo_configsvr1 --net=host --privileged --restart=always \
-e MONGO_INITDB_ROOT_USERNAME=root \
-e MONGO_INITDB_ROOT_PASSWORD=SYiXHC0hWE \
-v /data/docker_volumn/mongo_shard_cluster/configsvr1/data:/data/configdb \
mongo:4.4.3 \
numactl --interleave=all mongod --bind_ip 10.234.6.33 --port 27015 --replSet config --configsvr

# docker stop mongo_configsvr2 && docker rm mongo_configsvr2
docker run -d --name mongo_configsvr2 --net=host --privileged --restart=always \
-e MONGO_INITDB_ROOT_USERNAME=root \
-e MONGO_INITDB_ROOT_PASSWORD=SYiXHC0hWE \
-v /data/docker_volumn/mongo_shard_cluster/configsvr2/data:/data/configdb \
mongo:4.4.3 \
numactl --interleave=all mongod --bind_ip 10.234.6.33 --port 27025 --replSet config --configsvr

# docker stop mongo_configsvr3 && docker rm mongo_configsvr3
docker run -d --name mongo_configsvr3 --net=host --privileged --restart=always \
-e MONGO_INITDB_ROOT_USERNAME=root \
-e MONGO_INITDB_ROOT_PASSWORD=SYiXHC0hWE \
-v /data/docker_volumn/mongo_shard_cluster/configsvr3/data:/data/configdb \
mongo:4.4.3 \
numactl --interleave=all mongod --bind_ip 10.234.6.33 --port 27035 --replSet config --configsvr

# 初始化集群 configsvr
docker exec -it -e COLUMNS=200 -e LINES=200 mongo_configsvr1 mongo admin --port 27015 --host 10.234.6.33
> rs.initiate(
   {
      _id: "config",
      configsvr: true,
      members: [
         { _id: 0, host : "10.234.6.33:27015" },
         { _id: 1, host : "10.234.6.33:27025" },
         { _id: 2, host : "10.234.6.33:27035" }
      ]
   }
);
config:PRIMARY> rs.status();
config:PRIMARY> use admin
config:PRIMARY> db.createUser(
  {
    user: "root",
    pwd: "SYiXHC0hWE",
    roles: [
       { role: "root", db: "admin" }
    ]
  }
);

# 启用认证
docker stop mongo_configsvr1 && docker rm mongo_configsvr1
docker run -d --name mongo_configsvr1 --net=host --privileged --restart=always \
-e MONGO_INITDB_ROOT_USERNAME=root \
-e MONGO_INITDB_ROOT_PASSWORD=SYiXHC0hWE \
-v /data/docker_volumn/mongo_shard_cluster/configsvr1/data:/data/configdb \
mongo:4.4.3 \
numactl --interleave=all mongod --bind_ip 10.234.6.33 --port 27015 --replSet config --configsvr --auth --keyFile=/data/configdb/key_file
docker stop mongo_configsvr2 && docker rm mongo_configsvr2
docker run -d --name mongo_configsvr2 --net=host --privileged --restart=always \
-e MONGO_INITDB_ROOT_USERNAME=root \
-e MONGO_INITDB_ROOT_PASSWORD=SYiXHC0hWE \
-v /data/docker_volumn/mongo_shard_cluster/configsvr2/data:/data/configdb \
mongo:4.4.3 \
numactl --interleave=all mongod --bind_ip 10.234.6.33 --port 27025 --replSet config --configsvr --auth --keyFile=/data/configdb/key_file
docker stop mongo_configsvr3 && docker rm mongo_configsvr3
docker run -d --name mongo_configsvr3 --net=host --privileged --restart=always \
-e MONGO_INITDB_ROOT_USERNAME=root \
-e MONGO_INITDB_ROOT_PASSWORD=SYiXHC0hWE \
-v /data/docker_volumn/mongo_shard_cluster/configsvr3/data:/data/configdb \
mongo:4.4.3 \
numactl --interleave=all mongod --bind_ip 10.234.6.33 --port 27035 --replSet config --configsvr --auth --keyFile=/data/configdb/key_file

docker exec -it -e COLUMNS=200 -e LINES=200 mongo_configsvr1 mongo admin --port 27015 --host 10.234.6.33 -u root -p SYiXHC0hWE

# 初始化 mongos
mkdir -p /data/docker_volumn/mongo_shard_cluster/mongos1/data
mkdir -p /data/docker_volumn/mongo_shard_cluster/mongos2/data
mkdir -p /data/docker_volumn/mongo_shard_cluster/mongos3/data
\cp -f /data/docker_volumn/mongo_shard_cluster/shardsvr11/data/key_file /data/docker_volumn/mongo_shard_cluster/mongos1/data
\cp -f /data/docker_volumn/mongo_shard_cluster/shardsvr11/data/key_file /data/docker_volumn/mongo_shard_cluster/mongos2/data
\cp -f /data/docker_volumn/mongo_shard_cluster/shardsvr11/data/key_file /data/docker_volumn/mongo_shard_cluster/mongos3/data
ll /data/docker_volumn/mongo_shard_cluster/mongos1/data
ll /data/docker_volumn/mongo_shard_cluster/mongos2/data
ll /data/docker_volumn/mongo_shard_cluster/mongos3/data

# docker stop mongo_mongos1 && docker rm mongo_mongos1
docker run -d --name mongo_mongos1 --net=host --privileged --restart=always \
-e MONGO_INITDB_ROOT_USERNAME=root \
-e MONGO_INITDB_ROOT_PASSWORD=SYiXHC0hWE \
-v /data/docker_volumn/mongo_shard_cluster/mongos1/data:/data/configdb \
--entrypoint "mongos" \
mongo:4.4.3 \
--bind_ip 10.234.6.33 --port 27016 --configdb config/10.234.6.33:27015,10.234.6.33:27025,10.234.6.33:27035 \
--keyFile=/data/configdb/key_file

# docker stop mongo_mongos2 && docker rm mongo_mongos2
docker run -d --name mongo_mongos2 --net=host --privileged --restart=always \
-e MONGO_INITDB_ROOT_USERNAME=root \
-e MONGO_INITDB_ROOT_PASSWORD=SYiXHC0hWE \
-v /data/docker_volumn/mongo_shard_cluster/mongos2/data:/data/configdb \
--entrypoint "mongos" \
mongo:4.4.3 \
--bind_ip 10.234.6.33 --port 27026 --configdb config/10.234.6.33:27015,10.234.6.33:27025,10.234.6.33:27035 \
--keyFile=/data/configdb/key_file

# docker stop mongo_mongos3 && docker rm mongo_mongos3
docker run -d --name mongo_mongos3 --net=host --privileged --restart=always \
-e MONGO_INITDB_ROOT_USERNAME=root \
-e MONGO_INITDB_ROOT_PASSWORD=SYiXHC0hWE \
-v /data/docker_volumn/mongo_shard_cluster/mongos3/data:/data/configdb \
--entrypoint "mongos" \
mongo:4.4.3 \
--bind_ip 10.234.6.33 --port 27036 --configdb config/10.234.6.33:27015,10.234.6.33:27025,10.234.6.33:27035 \
--keyFile=/data/configdb/key_file

docker exec -it -e COLUMNS=200 -e LINES=200 mongo_mongos1 mongo admin --port 27016 --host 10.234.6.33 -u root -p SYiXHC0hWE
docker exec -it -e COLUMNS=200 -e LINES=200 mongo_mongos2 mongo admin --port 27026 --host 10.234.6.33 -u root -p SYiXHC0hWE
docker exec -it -e COLUMNS=200 -e LINES=200 mongo_mongos2 mongo admin --port 27036 --host 10.234.6.33 -u root -p SYiXHC0hWE

# 添加分片到集群
mongos> sh.addShard("shard1/10.234.6.33:27017,10.234.6.33:27027,10.234.6.33:27037");
mongos> sh.status();

# 启用分片
mongos> sh.enableSharding("db_test");
mongos> sh.shardCollection("db_test.order", {_id: "hashed"});

# 测试分片集群
mongos> use db_test
mongos> for (i = 1; i <= 1001; i=i+1){
db.order.insert({i: i})
}
mongos> sh.status();

# 部署集群 shard2
mkdir -p /data/docker_volumn/mongo_shard_cluster/shardsvr21/data
mkdir -p /data/docker_volumn/mongo_shard_cluster/shardsvr22/data
mkdir -p /data/docker_volumn/mongo_shard_cluster/shardsvr23/data
\cp -f /data/docker_volumn/mongo_shard_cluster/shardsvr11/data/key_file /data/docker_volumn/mongo_shard_cluster/shardsvr21/data/
\cp -f /data/docker_volumn/mongo_shard_cluster/shardsvr11/data/key_file /data/docker_volumn/mongo_shard_cluster/shardsvr22/data/
\cp -f /data/docker_volumn/mongo_shard_cluster/shardsvr11/data/key_file /data/docker_volumn/mongo_shard_cluster/shardsvr23/data/
ll /data/docker_volumn/mongo_shard_cluster/shardsvr21/data/key_file
ll /data/docker_volumn/mongo_shard_cluster/shardsvr22/data/key_file
ll /data/docker_volumn/mongo_shard_cluster/shardsvr23/data/key_file

# docker stop mongo_shardsvr21 && docker rm mongo_shardsvr21
docker run -d --name mongo_shardsvr21 --net=host --privileged --restart=always \
-v /data/docker_volumn/mongo_shard_cluster/shardsvr21/data:/data/db \
mongo:4.4.3 \
numactl --interleave=all mongod --bind_ip 10.234.6.33 --port 27018 --replSet shard2 --shardsvr

# docker stop mongo_shardsvr22 && docker rm mongo_shardsvr22
docker run -d --name mongo_shardsvr22 --net=host --privileged --restart=always \
-v /data/docker_volumn/mongo_shard_cluster/shardsvr22/data:/data/db \
mongo:4.4.3 \
numactl --interleave=all mongod --bind_ip 10.234.6.33 --port 27028 --replSet shard2 --shardsvr

# docker stop mongo_shardsvr23 && docker rm mongo_shardsvr23
docker run -d --name mongo_shardsvr23 --net=host --privileged --restart=always \
-v /data/docker_volumn/mongo_shard_cluster/shardsvr23/data:/data/db \
mongo:4.4.3 \
numactl --interleave=all mongod --bind_ip 10.234.6.33 --port 27038 --replSet shard2 --shardsvr

# 初始化集群 shard2
docker exec -it -e COLUMNS=200 -e LINES=200 mongo_shardsvr21 mongo admin --port 27018 --host 10.234.6.33
> rs.initiate(
   {
      _id: "shard2",
      members: [
         { _id: 0, host : "10.234.6.33:27018" },
         { _id: 1, host : "10.234.6.33:27028" },
         { _id: 2, host : "10.234.6.33:27038" }
      ]
   }
);
shard2:PRIMARY> rs.status();
shard2:PRIMARY> use admin
shard2:PRIMARY> db.createUser(
  {
    user: "root",
    pwd: "SYiXHC0hWE",
    roles: [
       { role: "root", db: "admin" }
    ]
  }
);

# 启用认证
docker stop mongo_shardsvr21 && docker rm mongo_shardsvr21
docker run -d --name mongo_shardsvr21 --net=host --privileged --restart=always \
-v /data/docker_volumn/mongo_shard_cluster/shardsvr21/data:/data/db \
mongo:4.4.3 \
numactl --interleave=all mongod --bind_ip 10.234.6.33 --port 27018 --replSet shard2 --shardsvr --auth --keyFile=/data/db/key_file
docker stop mongo_shardsvr22 && docker rm mongo_shardsvr22
docker run -d --name mongo_shardsvr22 --net=host --privileged --restart=always \
-v /data/docker_volumn/mongo_shard_cluster/shardsvr22/data:/data/db \
mongo:4.4.3 \
numactl --interleave=all mongod --bind_ip 10.234.6.33 --port 27028 --replSet shard2 --shardsvr --auth --keyFile=/data/db/key_file
docker stop mongo_shardsvr23 && docker rm mongo_shardsvr23
docker run -d --name mongo_shardsvr23 --net=host --privileged --restart=always \
-v /data/docker_volumn/mongo_shard_cluster/shardsvr23/data:/data/db \
mongo:4.4.3 \
numactl --interleave=all mongod --bind_ip 10.234.6.33 --port 27038 --replSet shard2 --shardsvr --auth --keyFile=/data/db/key_file

docker exec -it -e COLUMNS=200 -e LINES=200 mongo_shardsvr21 mongo admin --port 27018 --host 10.234.6.33 -u root -p SYiXHC0hWE
shard2:PRIMARY> rs.status();

# 添加分片到集群
docker exec -it -e COLUMNS=200 -e LINES=200 mongo_mongos1 mongo admin --port 27016 --host 10.234.6.33 -u root -p SYiXHC0hWE
mongos> sh.addShard("shard2/10.234.6.33:27018,10.234.6.33:27028,10.234.6.33:27038");
mongos> sh.status();

# 等待一小段时间，发现分片数据会自动重平衡
```

### 分片集群(高可用)

#### 集群架构

| 服务器IP    | 服务器域名 | 端口  | mongo角色     | 文件目录 |
| ----------- |---- | ----- | ------------- | ----------- |
| 10.234.6.33 | scm.logistics.01.mongodb.suzhou.belle.lan | 27015 | config-server | configsvr1/data |
|             |      | 27016 | mongos        | mongos1/data |
|             |      | 27017 | shard1        | shardsvr1/data |
|             |      | 27018 | shard2        | shardsvr1/data |
|             |      | 27019 | shard3        | shardsvr1/data |
| 10.234.6.206 | scm.logistics.02.mongodb.suzhou.belle.lan | 27015 | config-server | configsvr2/data |
|             |      | 27016 | mongos        | mongos2/data |
|             |      | 27017 | shard1        | shardsvr2/data |
|             |      | 27018 | shard2        | shardsvr2/data |
|             |      | 27019 | shard3        | shardsvr2/data |
| 10.234.8.142 | scm.logistics.03.mongodb.suzhou.belle.lan | 27015 | config-server | configsvr3/data |
|             |      | 27016 | mongos        | mongos3/data |
|             |      | 27017 | shard1        | shardsvr3/data |
|             |      | 27018 | shard2        | shardsvr3/data |
|             |      | 27019 | shard3        | shardsvr3/data |

#### 集群安装

```sh
# 1、定义域名

# 2、创建key(ssh 10.234.6.33 操作)
mkdir -p /data/docker_volumn/mongo_shard_cluster/auth
openssl rand -base64 756 > /data/docker_volumn/mongo_shard_cluster/auth/key_file
chmod 400 /data/docker_volumn/mongo_shard_cluster/auth/key_file
cat /data/docker_volumn/mongo_shard_cluster/auth/key_file
scp -r /data/docker_volumn/mongo_shard_cluster 10.234.6.206:/data/docker_volumn/
scp -r /data/docker_volumn/mongo_shard_cluster 10.234.8.142:/data/docker_volumn/

# 3、定义编排文件
```

编排文件：/data/mongodb/docker-compose.yml 

```yaml
# 官方网站： https://docs.docker.com/compose/compose-file/

# 筛选docker容器脚本
# docker ps | grep -v google | grep -v "bf/" | grep -v k8s

# scm 开发测试公共环境基础组件编排文件

version: '3'

services:

  mongo_configsvr:
    image: mongo:4.4.3
    container_name: mongo_configsvr
    restart: always
    network_mode: host
    privileged: true
    volumes:
    - /data/docker_volumn/mongo_shard_cluster/configsvr/data/db:/data/db
    - /data/docker_volumn/mongo_shard_cluster/configsvr/data/configdb:/data/configdb
    - /data/docker_volumn/mongo_shard_cluster/configsvr/data/backup:/data/backup
    - /data/docker_volumn/mongo_shard_cluster/configsvr/data/tmp:/data/tmp
    - /data/docker_volumn/mongo_shard_cluster/auth:/data/auth
    command: numactl --interleave=all mongod --bind_ip_all --port 27015 --replSet config --configsvr

  mongo_shardsvr1:
    image: mongo:4.4.3
    container_name: mongo_shardsvr1
    restart: always
    network_mode: host
    privileged: true
    volumes:
    - /data/docker_volumn/mongo_shard_cluster/shardsvr1/data/db:/data/db
    - /data/docker_volumn/mongo_shard_cluster/shardsvr1/data/configdb:/data/configdb
    - /data/docker_volumn/mongo_shard_cluster/shardsvr1/data/backup:/data/backup
    - /data/docker_volumn/mongo_shard_cluster/shardsvr1/data/tmp:/data/tmp
    - /data/docker_volumn/mongo_shard_cluster/auth:/data/auth
    command: numactl --interleave=all mongod --bind_ip_all --port 27017 --replSet shard1 --shardsvr
    depends_on:
    - mongo_configsvr

  mongo_shardsvr2:
    image: mongo:4.4.3
    container_name: mongo_shardsvr2
    restart: always
    network_mode: host
    privileged: true
    volumes:
    - /data/docker_volumn/mongo_shard_cluster/shardsvr2/data/db:/data/db
    - /data/docker_volumn/mongo_shard_cluster/shardsvr2/data/configdb:/data/configdb
    - /data/docker_volumn/mongo_shard_cluster/shardsvr2/data/backup:/data/backup
    - /data/docker_volumn/mongo_shard_cluster/shardsvr2/data/tmp:/data/tmp
    - /data/docker_volumn/mongo_shard_cluster/auth:/data/auth
    command: numactl --interleave=all mongod --bind_ip_all --port 27018 --replSet shard2 --shardsvr
    depends_on:
    - mongo_configsvr
    
  mongo_shardsvr3:
    image: mongo:4.4.3
    container_name: mongo_shardsvr3
    restart: always
    network_mode: host
    privileged: true
    volumes:
    - /data/docker_volumn/mongo_shard_cluster/shardsvr3/data/db:/data/db
    - /data/docker_volumn/mongo_shard_cluster/shardsvr3/data/configdb:/data/configdb
    - /data/docker_volumn/mongo_shard_cluster/shardsvr3/data/backup:/data/backup
    - /data/docker_volumn/mongo_shard_cluster/shardsvr3/data/tmp:/data/tmp
    - /data/docker_volumn/mongo_shard_cluster/auth:/data/auth
    command: numactl --interleave=all mongod --bind_ip_all --port 27019 --replSet shard3 --shardsvr
    depends_on:
    - mongo_configsvr

  mongo_mongos:
    image: mongo:4.4.3
    container_name: mongo_mongos
    restart: always
    network_mode: host
    privileged: true
    volumes:
    - /data/docker_volumn/mongo_shard_cluster/mongos/data/db:/data/db
    - /data/docker_volumn/mongo_shard_cluster/mongos/data/configdb:/data/configdb
    - /data/docker_volumn/mongo_shard_cluster/mongos/data/backup:/data/backup
    - /data/docker_volumn/mongo_shard_cluster/mongos/data/tmp:/data/tmp
    - /data/docker_volumn/mongo_shard_cluster/auth:/data/auth
    entrypoint: mongos
    command: --bind_ip_all --port 27016 --configdb config/scm.logistics.01.mongodb.suzhou.belle.lan:27015,scm.logistics.02.mongodb.suzhou.belle.lan:27015,scm.logistics.03.mongodb.suzhou.belle.lan:27015
    depends_on:
    - mongo_shardsvr1
    - mongo_shardsvr2
```




```sh
# 4、启动容器
cd /data/mongodb
docker-compose up -d 

# 初始化集群 configsvr
docker exec -it -e COLUMNS=200 -e LINES=200 mongo_configsvr mongo admin --port 27015 --host scm.logistics.01.mongodb.suzhou.belle.lan
> rs.initiate(
   {
      _id: "config",
      configsvr: true,
      members: [
         { _id: 0, host : "scm.logistics.01.mongodb.suzhou.belle.lan:27015" },
         { _id: 1, host : "scm.logistics.02.mongodb.suzhou.belle.lan:27015" },
         { _id: 2, host : "scm.logistics.03.mongodb.suzhou.belle.lan:27015" }
      ]
   }
);
config:PRIMARY> rs.status();
config:PRIMARY> use admin
config:PRIMARY> db.createUser(
  {
    user: "root",
    pwd: "SYiXHC0hWE",
    roles: [
       { role: "root", db: "admin" }
    ]
  }
);


# 初始化集群 shard1
docker exec -it -e COLUMNS=200 -e LINES=200 mongo_shardsvr1 mongo admin --port 27017 --host scm.logistics.01.mongodb.suzhou.belle.lan
> rs.initiate(
   {
      _id: "shard1",
      members: [
         { _id: 0, host : "scm.logistics.01.mongodb.suzhou.belle.lan:27017" },
         { _id: 1, host : "scm.logistics.02.mongodb.suzhou.belle.lan:27017" },
         { _id: 2, host : "scm.logistics.03.mongodb.suzhou.belle.lan:27017" }
      ]
   }
);
shard1:PRIMARY> rs.status();
shard1:PRIMARY> use admin
shard1:PRIMARY> db.createUser(
  {
    user: "root",
    pwd: "SYiXHC0hWE",
    roles: [
       { role: "root", db: "admin" }
    ]
  }
);

# 初始化集群 shard2
docker exec -it -e COLUMNS=200 -e LINES=200 mongo_shardsvr2 mongo admin --port 27018 --host scm.logistics.01.mongodb.suzhou.belle.lan
> rs.initiate(
   {
      _id: "shard2",
      members: [
         { _id: 0, host : "scm.logistics.01.mongodb.suzhou.belle.lan:27018" },
         { _id: 1, host : "scm.logistics.02.mongodb.suzhou.belle.lan:27018" },
         { _id: 2, host : "scm.logistics.03.mongodb.suzhou.belle.lan:27018" }
      ]
   }
);
shard1:PRIMARY> rs.status();
shard1:PRIMARY> use admin
shard1:PRIMARY> db.createUser(
  {
    user: "root",
    pwd: "SYiXHC0hWE",
    roles: [
       { role: "root", db: "admin" }
    ]
  }
);

# 初始化集群 shard3
docker exec -it -e COLUMNS=200 -e LINES=200 mongo_shardsvr2 mongo admin --port 27019 --host scm.logistics.01.mongodb.suzhou.belle.lan
> rs.initiate(
   {
      _id: "shard3",
      members: [
         { _id: 0, host : "scm.logistics.01.mongodb.suzhou.belle.lan:27019" },
         { _id: 1, host : "scm.logistics.02.mongodb.suzhou.belle.lan:27019" },
         { _id: 2, host : "scm.logistics.03.mongodb.suzhou.belle.lan:27019" }
      ]
   }
);
shard1:PRIMARY> rs.status();
shard1:PRIMARY> use admin
shard1:PRIMARY> db.createUser(
  {
    user: "root",
    pwd: "SYiXHC0hWE",
    roles: [
       { role: "root", db: "admin" }
    ]
  }
);

# 初始化 mongos
docker exec -it -e COLUMNS=200 -e LINES=200 mongo_mongos mongo admin --port 27016 --host scm.logistics.01.mongodb.suzhou.belle.lan

# 添加分片到集群
mongos> sh.addShard("shard1/scm.logistics.01.mongodb.suzhou.belle.lan:27017,scm.logistics.02.mongodb.suzhou.belle.lan:27017,scm.logistics.03.mongodb.suzhou.belle.lan:27017");
mongos> sh.addShard("shard2/scm.logistics.01.mongodb.suzhou.belle.lan:27018,scm.logistics.02.mongodb.suzhou.belle.lan:27018,scm.logistics.03.mongodb.suzhou.belle.lan:27018");
mongos> sh.addShard("shard3/scm.logistics.01.mongodb.suzhou.belle.lan:27019,scm.logistics.02.mongodb.suzhou.belle.lan:27019,scm.logistics.03.mongodb.suzhou.belle.lan:27019");
mongos> sh.status();


# 启用分片
mongos> sh.enableSharding("db_test");
mongos> sh.shardCollection("db_test.order", {_id: "hashed"});

# 测试分片集群
mongos> use db_test
mongos> for (i = 1; i <= 1001; i=i+1){
db.order.insert({i: i})
}
mongos> sh.status();
mongos> sh.getBalancerState()

# 等待一小段时间，发现分片数据会自动重平衡
```

开启认证

```yaml
# 官方网站： https://docs.docker.com/compose/compose-file/

# 筛选docker容器脚本
# docker ps | grep -v google | grep -v "bf/" | grep -v k8s

# scm 开发测试公共环境基础组件编排文件

version: '3'

services:

  mongo_configsvr:
    image: mongo:4.4.3
    container_name: mongo_configsvr
    restart: always
    network_mode: host
    privileged: true
    volumes:
    - /data/docker_volumn/mongo_shard_cluster/configsvr/data/db:/data/db
    - /data/docker_volumn/mongo_shard_cluster/configsvr/data/configdb:/data/configdb
    - /data/docker_volumn/mongo_shard_cluster/configsvr/data/backup:/data/backup
    - /data/docker_volumn/mongo_shard_cluster/configsvr/data/tmp:/data/tmp
    - /data/docker_volumn/mongo_shard_cluster/auth:/data/auth
    command: numactl --interleave=all mongod --bind_ip_all --port 27015 --replSet config --configsvr --auth --keyFile=/data/auth/key_file

  mongo_shardsvr1:
    image: mongo:4.4.3
    container_name: mongo_shardsvr1
    restart: always
    network_mode: host
    privileged: true
    volumes:
    - /data/docker_volumn/mongo_shard_cluster/shardsvr1/data/db:/data/db
    - /data/docker_volumn/mongo_shard_cluster/shardsvr1/data/configdb:/data/configdb
    - /data/docker_volumn/mongo_shard_cluster/shardsvr1/data/backup:/data/backup
    - /data/docker_volumn/mongo_shard_cluster/shardsvr1/data/tmp:/data/tmp
    - /data/docker_volumn/mongo_shard_cluster/auth:/data/auth
    command: numactl --interleave=all mongod --bind_ip_all --port 27017 --replSet shard1 --shardsvr --auth --keyFile=/data/auth/key_file
    depends_on:
    - mongo_configsvr

  mongo_shardsvr2:
    image: mongo:4.4.3
    container_name: mongo_shardsvr2
    restart: always
    network_mode: host
    privileged: true
    volumes:
    - /data/docker_volumn/mongo_shard_cluster/shardsvr2/data/db:/data/db
    - /data/docker_volumn/mongo_shard_cluster/shardsvr2/data/configdb:/data/configdb
    - /data/docker_volumn/mongo_shard_cluster/shardsvr2/data/backup:/data/backup
    - /data/docker_volumn/mongo_shard_cluster/shardsvr2/data/tmp:/data/tmp
    - /data/docker_volumn/mongo_shard_cluster/auth:/data/auth
    command: numactl --interleave=all mongod --bind_ip_all --port 27018 --replSet shard2 --shardsvr --auth --keyFile=/data/auth/key_file
    depends_on:
    - mongo_configsvr

  mongo_shardsvr3:
    image: mongo:4.4.3
    container_name: mongo_shardsvr3
    restart: always
    network_mode: host
    privileged: true
    volumes:
    - /data/docker_volumn/mongo_shard_cluster/shardsvr3/data/db:/data/db
    - /data/docker_volumn/mongo_shard_cluster/shardsvr3/data/configdb:/data/configdb
    - /data/docker_volumn/mongo_shard_cluster/shardsvr3/data/backup:/data/backup
    - /data/docker_volumn/mongo_shard_cluster/shardsvr3/data/tmp:/data/tmp
    - /data/docker_volumn/mongo_shard_cluster/auth:/data/auth
    command: numactl --interleave=all mongod --bind_ip_all --port 27019 --replSet shard3 --shardsvr --auth --keyFile=/data/auth/key_file
    depends_on:
    - mongo_configsvr

  mongo_mongos:
    image: mongo:4.4.3
    container_name: mongo_mongos
    restart: always
    network_mode: host
    privileged: true
    volumes:
    - /data/docker_volumn/mongo_shard_cluster/mongos/data/db:/data/db
    - /data/docker_volumn/mongo_shard_cluster/mongos/data/configdb:/data/configdb
    - /data/docker_volumn/mongo_shard_cluster/mongos/data/backup:/data/backup
    - /data/docker_volumn/mongo_shard_cluster/mongos/data/tmp:/data/tmp
    - /data/docker_volumn/mongo_shard_cluster/auth:/data/auth
    entrypoint: mongos
    command: --bind_ip_all --port 27016 --configdb config/scm.logistics.01.mongodb.suzhou.belle.lan:27015,scm.logistics.02.mongodb.suzhou.belle.lan:27015,scm.logistics.03.mongodb.suzhou.belle.lan:27015 --keyFile=/data/auth/key_file
    depends_on:
    - mongo_shardsvr1
    - mongo_shardsvr2

```



```sh
# 验证权限
docker exec -it -e COLUMNS=200 -e LINES=200 mongo_configsvr mongo admin --port 27015 --host scm.logistics.01.mongodb.suzhou.belle.lan -u root -p SYiXHC0hWE

docker exec -it -e COLUMNS=200 -e LINES=200 mongo_shardsvr1 mongo admin --port 27017 --host scm.logistics.01.mongodb.suzhou.belle.lan -u root -p SYiXHC0hWE

docker exec -it -e COLUMNS=200 -e LINES=200 mongo_shardsvr2 mongo admin --port 27018 --host scm.logistics.01.mongodb.suzhou.belle.lan -u root -p SYiXHC0hWE

docker exec -it -e COLUMNS=200 -e LINES=200 mongo_shardsvr3 mongo admin --port 27019 --host scm.logistics.01.mongodb.suzhou.belle.lan -u root -p SYiXHC0hWE

docker exec -it -e COLUMNS=200 -e LINES=200 mongo_mongos mongo admin --port 27016 --host scm.logistics.01.mongodb.suzhou.belle.lan -u root -p SYiXHC0hWE

# 创建普通账号
mongos> use db_test
mongos> db.createUser(
  {
    user: "user_test",
    pwd: "123456",
    roles: [
       { role: "readWrite", db: "db_test" }
    ]
  }
);

# 新增管理员账号，需要分别在mongo_configsvr、mongo_shardsvr1、mongo_shardsvr2、mongo_shardsvr3执行
use admin
db.createUser(
  {
    user: "liu.jiang",
    pwd: "8tB9s2esaV",
    roles: [
       { role: "root", db: "admin" }
    ]
  }
);
```

在分片集群中, 每个db都有一个主分片, 用来存储这个db中所有未分片的collection. 每个db都有各自的主分片. 主分片与副本集中的主副本无关.
mongos在创建新db时, 会选择集群中数据量最小的那个分片, 在其上创建主分片. 使用listDatabase命令返回的totalSize也是判断的一个因素. 
要修改db的主分片, 使用movePrimary命令. 迁移主分片的过程会明显耗时, 迁移的过程中不应该去访问对应的数据. 

```sh
# 查看数据库信息
mongos> db.adminCommand( { listDatabases: 1 } )
# 修改db的主分片
mongos> db.adminCommand( { movePrimary: "db_test2", to: "shard1" } )
mongos> db.adminCommand( { movePrimary: "db_test3", to: "shard1" } )

```



#### 参考资料

[MongoDB 分片集群技术](https://www.cnblogs.com/clsn/p/8214345.html)

[MongoDB笔记: 分片集群](https://www.cnblogs.com/milton/p/11466682.html)

[在Docker上部署mongodb分片副本集群](https://www.cnblogs.com/hehexiaoxia/p/6192796.html)

[Docker搭建MongoDB集群（副本分片）](https://www.cnblogs.com/mergy/p/12916517.html)

[docker方式部署mongodb带认证的分片副本集群](https://blog.csdn.net/guan0005/article/details/86995019)

[docker-compose搭建mongodb分片集群及安全身份认证（实战）](https://blog.csdn.net/yufei_java/article/details/103704582)

[MongoDB Cluster 数据平衡优化](https://www.cnblogs.com/xibuhaohao/p/13156722.html)

[MongoDB管理之分片集群实践](https://www.cnblogs.com/wangsicongde/p/7588632.html)

[MongoDB学习分片](https://www.cnblogs.com/liujitao79/p/6899793.html)

## 参考资料

[MongoDB的Replica Set以及Auth的配置](https://www.cnblogs.com/pbblogs/p/9783964.html)

[Enforce Keyfile Access Control in a Replica Set](https://docs.mongodb.com/manual/tutorial/enforce-keyfile-access-control-in-existing-replica-set/)

[Enforce Keyfile Access Control in a Replica Set without Downtime](https://docs.mongodb.com/manual/tutorial/enforce-keyfile-access-control-in-existing-replica-set-without-downtime/)

