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