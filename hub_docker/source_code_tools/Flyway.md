# Flyway

[https://flywaydb.org/](https://flywaydb.org/)

[flyway hub_docker](https://hub.docker.com/r/flyway/flyway)

[flyway documentation](https://flywaydb.org/documentation/)

### 云端数据库Spawn

[https://spawn.cc/](https://spawn.cc/)

[安装 Spawn](https://docs.spawn.cc/getting-started/installation)

这里就不安装了，使用本地的MySQL数据库。

### docker运行

```sh
# 查看帮助文件
docker run --rm hub.wonhigh.cn/library/flyway:7.10.0

docker run --rm hub.wonhigh.cn/library/flyway:7.10.0 \
-url=jdbc:MysqL://10.0.30.43:3306/db_flyway_test?useUnicode=true&characterEncoding=utf8&useSSL=false \
-user=root \
-password=Root123 \
info
```