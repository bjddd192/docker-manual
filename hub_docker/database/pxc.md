# mysql

## 官方镜像

[Hub官方](https://hub.docker.com/r/percona/percona-xtradb-cluster)

[官方文档](https://www.percona.com/doc/percona-xtradb-cluster/5.7/index.html)

[Installing Percona XtraDB Cluster](https://www.percona.com/doc/percona-xtradb-cluster/5.7/install/index.html)

[Running Percona XtraDB Cluster in a Docker Container](https://www.percona.com/doc/percona-xtradb-cluster/5.7/install/docker.html#pxc-docker-container-running)

## 启动命令

```sh
docker stop node1 && docker rm -f node1
docker stop node2 && docker rm -f node2
docker stop node3 && docker rm -f node3
docker network rm pxc-network  

docker network create --subnet=192.168.110.0/24 pxc-network

docker volume rm v1
docker volume create v1

docker run --name=node1 -d -p 3306:3306 \
    -e MYSQL_ROOT_PASSWORD=root \
    -e CLUSTER_NAME=mycluster \
    -e XTRABACKUP_PASSWORD=root \
    -v v1:/var/lib/mysql \
    --net=pxc-network --ip 192.168.110.111 \
    percona/percona-xtradb-cluster:5.7.27

# 切记因第一个节点初始化比较耗时一定要等第一个容器创建成功可以使用MySQL客户端连接了才能创建第二个，或者会报错创建不了下面的容器！！！

docker volume rm v2
docker volume create v2
    
docker run --name node2 -d -p 3307:3306 --privileged=true \
    -e MYSQL_ROOT_PASSWORD=root \
    -e CLUSTER_NAME=mycluster \
    -e XTRABACKUP_PASSWORD=root \
    -e CLUSTER_JOIN=node1 \
    -v v2:/var/lib/mysql \
    --net=pxc-network --ip 192.168.110.112 \
    percona/percona-xtradb-cluster:5.7.27
         
docker volume rm v3
docker volume create v3
    
docker run --name node3 -d -p 3308:3306 --privileged=true \
    -e MYSQL_ROOT_PASSWORD=root \
    -e CLUSTER_NAME=mycluster \
    -e XTRABACKUP_PASSWORD=root \
    -e CLUSTER_JOIN=node1 \
    -v v3:/var/lib/mysql \
    --net=pxc-network --ip 192.168.110.113 \
    percona/percona-xtradb-cluster:5.7.27
```

以上是简单使用初始化脚本，如正式使用，持久化还需要下下功夫。

## 异常处理

进入容器后使用mysql指令出现异常 unknown option --ck；
需要先修改容器内 mysql 配置文件 node.cnf(/etc/mysql/node.cnf）最后的 ck 字母

开启远程连接：

```sql
use mysql
grant all privileges on *.* to root@'%' identified by 'root' with grant option;
flush privileges;
```

**实操发现正常安装不会出现以上异常。**

## 参考资料

[docker安装（PXC）mysql集群](https://www.jianshu.com/p/e7d4eaaecbec)

[Docker搭建MySQL的PXC集群](https://www.cnblogs.com/lonelyxmas/p/10584902.html)

[PXC（mysql集群） docker重启失败异常](https://blog.csdn.net/qq_35394891/article/details/83065193)
