# ProxySQL

[官网](https://proxysql.com)

[官方文档](https://proxysql.com/documentation/)

[ProxySQL-Configuration](ProxySQL-Configuration)

[github](https://github.com/sysown/proxysql)

[dockerhub](https://hub.docker.com/r/proxysql/proxysql)

### 安装部署

#### 容器化部署

##### mysql主从容器部署

```sh
# docker stop mysql_master && docker rm mysql_master
docker run --name mysql_master -d -p 3306:3306 \
-e MYSQL_ALLOW_EMPTY_PASSWORD=1 \
-e TZ=Asia/Shanghai \
bergerx/mysql-replication:5.7 \
--character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

# docker stop mysql_slave && docker rm mysql_slave
docker run --name mysql_slave -d -p 3307:3306 \
-e MYSQL_ALLOW_EMPTY_PASSWORD=1 \
-e TZ=Asia/Shanghai \
-e MASTER_HOST=10.10.217.70 \
-e MASTER_PORT=3306 \
bergerx/mysql-replication:5.7 \
--character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

##### 初始化数据库账号

```sql
grant all privileges on *.* to monitor@'%' identified by 'monitor';
grant all privileges on *.* to user_app@'%' identified by 'user_app';

-- 从库设置为只读
set global read_only = 1
```

##### proxysql容器部署

```sh
mkdir -p /data/docker_volumn/proxysql
# 准备配置文件
tee /data/docker_volumn/proxysql/proxysql.cnf <<-'EOF'
datadir="/var/lib/proxysql"

# 控制管理界面功能的全局变量
admin_variables=
{
    admin_credentials="admin:admin;radmin:radmin"
    mysql_ifaces="0.0.0.0:6032"
}

# 控制处理传入 MySQL 流量的功能的全局变量
mysql_variables=
{
    threads=4
    max_connections=2048
    default_query_delay=0
    default_query_timeout=36000000
    have_compress=true
    poll_timeout=2000
    interfaces="0.0.0.0:6033"
    default_schema="information_schema"
    stacksize=1048576
    server_version="5.5.30"
    connect_timeout_server=3000
    monitor_username="monitor"
    monitor_password="monitor"
    monitor_history=600000
    monitor_connect_interval=60000
    monitor_ping_interval=10000
    monitor_read_only_interval=1500
    monitor_read_only_timeout=500
    ping_interval_server_msec=120000
    ping_timeout_server=500
    commands_stats=true
    sessions_sort=true
    connect_retries_on_failure=10
}

# defines all the MySQL servers
mysql_servers =
(
  { address="10.10.217.70"
    port=3306
    hostgroup=1
    max_connections=2000
  },
  { address="10.10.217.70"
	port=3307
	hostgroup=2
	max_connections=2000
  }
)

# defines all the MySQL users
mysql_users:
(
  {
    username = "user_app" 
    password = "user_app"
    default_hostgroup = 1
    max_connections=1000
    default_schema="test"
    active = 1 
  }
)

#defines MySQL Query Rules
mysql_query_rules:
(
  {
    rule_id=1
    active=1
    match_pattern="^SELECT .* FOR UPDATE$"
    destination_hostgroup=0
    apply=1
  },
  {
    rule_id=2
    active=1
    match_pattern="^SELECT"
    destination_hostgroup=1
    apply=1
  }
)
EOF

# docker stop proxysql && docker rm proxysql
docker run --name proxysql -d -p 16032:6032 -p 16033:6033 -p 16070:6070 --restart=always \
-v /data/docker_volumn/proxysql/proxysql.cnf:/etc/proxysql.cnf \
proxysql/proxysql:2.3.2

# 查看版本
docker exec -it proxysql proxysql --version

# 远程登录 ProxySQL
mysql -h127.0.0.1 -P16032 -uradmin -pradmin --prompt "ProxySQL Admin > "
```

##### proxysql常用操作

```sql
-- 常用查询
SELECT * FROM mysql_servers;
SELECT * FROM mysql_replication_hostgroups;
SELECT * FROM mysql_query_rules;
SELECT * FROM global_variables WHERE variable_name LIKE 'mysql-monitor_%';
SHOW TABLES FROM stats;
SELECT * FROM stats.stats_mysql_connection_pool;
SELECT * FROM stats_mysql_commands_counters WHERE Total_cnt;
SELECT hostgroup hg, sum_time, count_star, digest_text FROM stats_mysql_query_digest ORDER BY sum_time DESC;


-- 添加后端
-- ProxySQL 认为后端实例具有 read_only = 0 作为 WRITER 实例，所以这应该只在你的主 MySQL 服务器上设置
-- 从 MySQL 服务器设置 read_only = 1
INSERT INTO mysql_servers(hostgroup_id,hostname,port) VALUES (1,'10.10.217.70',3306);
INSERT INTO mysql_servers(hostgroup_id,hostname,port) VALUES (1,'10.10.217.70',3307);

-- 配置监控
-- ProxySQL 持续监控 MySQL 服务器后端配置以识别健康状态。
UPDATE global_variables SET variable_value='monitor' WHERE variable_name='mysql-monitor_username';
UPDATE global_variables SET variable_value='monitor' WHERE variable_name='mysql-monitor_password';
UPDATE global_variables SET variable_value='2000' WHERE variable_name IN 
('mysql-monitor_connect_interval','mysql-monitor_ping_interval','mysql-monitor_read_only_interval');
SELECT * FROM global_variables WHERE variable_name LIKE 'mysql-monitor_%';
-- 保持配置更改
LOAD MYSQL VARIABLES TO RUNTIME;
SAVE MYSQL VARIABLES TO DISK;

-- 后台健康检查
SHOW TABLES FROM monitor;
SELECT * FROM monitor.mysql_server_connect_log ORDER BY time_start_us DESC LIMIT 3;
SELECT * FROM monitor.mysql_server_ping_log ORDER BY time_start_us DESC LIMIT 3;

-- 在验证服务器被正确监控并且运行正常后，激活配置
LOAD MYSQL SERVERS TO RUNTIME;
SELECT * FROM mysql_servers;

-- MySQL 复制服务器分组 
SHOW CREATE TABLE mysql_replication_hostgroups\G
INSERT INTO mysql_replication_hostgroups (writer_hostgroup,reader_hostgroup,comment) VALUES (1,2,'cluster1');
LOAD MYSQL SERVERS TO RUNTIME;
SELECT * FROM monitor.mysql_server_read_only_log ORDER BY time_start_us DESC LIMIT 3;
SELECT * FROM mysql_servers;

-- 保持配置更改
SAVE MYSQL SERVERS TO DISK;
SAVE MYSQL VARIABLES TO DISK;

-- 配置mysql用户
INSERT INTO mysql_users(username,password,default_hostgroup) VALUES ('root','',1);
INSERT INTO mysql_users(username,password,default_hostgroup) VALUES ('user_app','user_app',1);
SELECT * FROM mysql_users;
LOAD MYSQL USERS TO RUNTIME;
SAVE MYSQL USERS TO DISK;
-- 测试连接
mysql -uuser_app -puser_app -h 10.10.217.70 -P16033 -e "SELECT @@port";

-- 查询规则配置
SHOW CREATE TABLE mysql_query_rules\G
set mysql-set_query_lock_on_hostgroup=0;
LOAD MYSQL VARIABLES TO RUNTIME;
SAVE MYSQL VARIABLES TO DISK;
-- 禁止使用没有WHERE的UPDATE
INSERT INTO mysql_query_rules (rule_id,active,match_digest,destination_hostgroup,apply,flagOUT) 
VALUES (10,1,'^\s*UPDATE (?!.[\s\S]*(where))',1,0,403);
-- 禁止使用没有WHERE的DELETE
INSERT INTO mysql_query_rules (rule_id,active,match_digest,destination_hostgroup,apply,flagOUT) 
VALUES (20,1,'^\s*DELETE (?!.[\s\S]*(where))',1,0,403);
-- 强制走写库
INSERT INTO mysql_query_rules (rule_id,active,match_digest,destination_hostgroup,apply) 
VALUES (30,1,'^\s*/\*master\*/',1,1);
-- SELECT * FROM user FOR UPDATE强制走写库
INSERT INTO mysql_query_rules (rule_id,active,match_digest,destination_hostgroup,apply) 
VALUES (40,1,'^\s*SELECT [\s\S]* FOR UPDATE$',1,1);
-- 默认所有SELECT走从库
INSERT INTO mysql_query_rules (rule_id,active,match_digest,destination_hostgroup,apply) 
VALUES (50,1,'^\s*SELECT',2,1);
-- 通用行为
INSERT INTO mysql_query_rules (rule_id,active,apply,flagIN,error_msg) 
VALUES (9999,1,1,403,'Query not allowed');
LOAD MYSQL QUERY RULES TO RUNTIME;
SAVE MYSQL QUERY RULES TO DISK;
```

### 异常处理

[roxySQL Error: connection is locked to hostgroup](https://github.com/sysown/proxysql/issues/2522)

### 参考资料

[proxysql的安装和使用(docker)](https://blog.csdn.net/song_java/article/details/87804083?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ELandingCtr%7Edefault-3.queryctr&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ELandingCtr%7Edefault-3.queryctr&utm_relevant_index=6)
