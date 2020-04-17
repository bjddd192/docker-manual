# FineBI

FineBI 是新一代自助大数据分析的 BI 软件。

[官网](https://www.finebi.com/)

[Hub官方](https://hub.docker.com/r/haodong/finebi)

[Github官方](https://github.com/haodong/docker-finebi)

[FineBI帮助文档](https://help.finebi.com/)

FineBI 激活码：	ca37747f-71501c812-d34c-e9871871a22a

## 安装包下载

```sh
wget https://fine-build.oss-cn-shanghai.aliyuncs.com/finebi/5.1/public/exe/spider/linux_unix_FineBI5_1-CN.sh
```

## 容器化安装

```sh
docker stop finebi && docker rm finebi

docker run -d --name finebi -p 31199:37799 --restart=always \
  haodong/finebi:version-5.1-2019.1.15

# 持久化数据
docker cp finebi:/usr/local/FineBI5.1 /data/docker_volumn/FineBI5.1

docker stop finebi && docker rm finebi

docker run -d --name finebi -p 31199:37799 --restart=always \
  -v /data/docker_volumn/FineBI5.1:/usr/local/FineBI5.1 \
  haodong/finebi:version-5.1-2019.1.15
```

镜像比较大，启动比较慢，需要耐心等待。

测试地址：

http://10.0.43.13:31199/webroot/decision

## 内部容器化安装

```sh
docker stop finebi && docker rm finebi

docker run -d --name finebi -p 62100:37799 --restart=always \
  --dns=10.0.43.27 \
  hub.wonhigh.cn/tools/finebi:version-5.1-2020.04.05
  
# 持久化数据
docker cp finebi:/usr/local/FineBI5.1 /data/docker_volumn/FineBI5.1

docker stop finebi && docker rm finebi

docker run -d --name finebi -p 62100:37799 --restart=always \
  -v /data/docker_volumn/FineBI5.1:/usr/local/FineBI5.1 \
  --dns=10.0.43.27 \
  hub.wonhigh.cn/tools/finebi:version-5.1-2020.04.05
```

## 常用目录

```sh
# 内存参数文件（默认为4G）
cat /usr/local/FineBI5.1/bin/finebi.vmoptions

# 日志文件
tail -f /usr/local/FineBI5.1/logs/fanruan.log
```

## 启用外部数据库

mysql 启用外部数据库有问题，用 oracle OK。

```sql
create database db_finedb DEFAULT CHARSET utf8 COLLATE utf8_bin;
GRANT ALL PRIVILEGES ON `db_finedb`.* TO 'user_finedb'@'%' IDENTIFIED by 'scm_finedb';
flush PRIVILEGES;

CREATE TABLE FINE_EXCEL_SUBMIT_TASK (
	ID VARCHAR(255) NOT NULL,
	CREATETIME TIMESTAMP NOT NULL,
	DESCRIPTION VARCHAR(1000),
	NAME VARCHAR(255) NOT NULL,
	REPORTPATH VARCHAR(1000) NOT NULL,
	SUBMITTIME TIMESTAMP,
	PRIMARY KEY (ID)
);
CREATE UNIQUE INDEX idx_NAME ON FINE_EXCEL_SUBMIT_TASK (NAME);
```

## 实施步骤

```sh
1. nginx 配置，增加了 https、websocket 支持 
2. 数据持久化
3. 内存参数优化(需重启)
4. 邮箱设置
5. 日志级别设置(无需重启)
6. 配置外接数据库(使用oracle数据库)
```

## 视频教程

[BI工程师 - 入门指南](https://bbs.fanruan.com/lesson-936.html)

## 测试库

```sql
finebi
jdbc:mysql://solutions.finebi.com:3306/finebi
finebi/finebi
```
