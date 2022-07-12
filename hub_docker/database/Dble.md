# Dble

[官方网站](https://opensource.actionsky.com/)

[github](https://github.com/actiontech/dble)

[gitlab官方文档(推荐)](https://github.com/actiontech/dble-docs-cn/blob/3.21.10.0/tag/SUMMARY.md)

[官方文档](https://actiontech.github.io/dble-docs-cn/)

[dockerhub](https://hub.docker.com/r/actiontech/dble)

[DBLE系列公开课](https://ke.qq.com/course/1586564?taid=7337384691119492)

### 安装部署

#### docker-compose 部署

```yaml
version: '2'
networks:
    net:
        driver: bridge
        ipam:
            config:
                - subnet: 172.18.0.0/16
                  gateway: 172.18.0.253
services:
    mysql1:
        image: mysql:8.0.13
        container_name: backend-mysql1
        hostname: backend-mysql1
        privileged: true
        stdin_open: true
        tty: true
        ports:
            - "33061:3306"
        networks:
            net:
              ipv4_address: 172.18.0.2
        environment:
            MYSQL_ROOT_PASSWORD: 123456
    mysql2:
        image: mysql:8.0.13
        container_name: backend-mysql2
        hostname: backend-mysql2
        privileged: true
        stdin_open: true
        tty: true
        ports:
            - "33062:3306"
        networks:
            net:
              ipv4_address: 172.18.0.3
        environment:
            MYSQL_ROOT_PASSWORD: 123456
    dble-server:
        image: actiontech/dble:3.21.10.0
        container_name: dble-server
        hostname: dble-server
        privileged: true
        stdin_open: true
        tty: true
        command: ["/opt/dble/bin/wait-for-it.sh", "backend-mysql1:3306","--","/opt/dble/bin/docker_init_start.sh"]
        ports:
            - "8066:8066"
            - "9066:9066"
        depends_on:
            - "mysql1"
            - "mysql2"
        networks:
            net:
              ipv4_address: 172.18.0.5
```

### 常用管理指令

```sh
# 登录管理控制台
mysql -h10.0.43.13 -uman1 -p654321 -P9066
mysql> use dble_information
mysql> show tables;
# 重新加载配置
mysql> reload @@config;
mysql> reload @@config_all;
mysql> reload @@metadata;
# 查看线程池
mysql> show @@threadpool;
mysql> show @@threadpool.task;
mysql> show @@thread_used;

# 查看dble版本
select * from dble_information.version;
# 查看dble的参数
select * from dble_information.dble_variables;
# 查看dble的运行状态
select * from dble_status;
select * from dble_reload_status;
show @@reload_status;
# 查看dble线程池
select * from dble_thread_pool;
select * from dble_thread_usage;
show @@thread_used;
# 查看dble线程池任务消费情况
select * from dble_thread_pool_task;
# 查看dble处理器状态
select * from dble_processor;
# 查看dble配置信息
select * from dble_sharding_node;
select * from dble_db_group;
select * from dble_db_instance;
select * from dble_schema;
select * from session_variables;
select * from session_connections;
select * from backend_variables;
select * from backend_connections;
select * from dble_table;
select * from dble_global_table;
select * from dble_sharding_table;
select * from dble_table_sharding_node;
# 查看拆分规则
select * from dble_algorithm;
# 查看用户信息
select * from dble_entry;
# 查看用户权限信息
select * from dble_entry_schema;
select * from dble_rw_split_entry;
select * from dble_entry_table_privilege;
select * from dble_blacklist;
select * from processlist;
show @@processlist;
```

### 参考资料

[https://blog.csdn.net/bingluo8787/article/details/100958104](使用Prometheus监控DBLE)
