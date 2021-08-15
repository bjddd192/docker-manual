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

### 默认占位符

[placeholder](https://flywaydb.org/documentation/configuration/placeholder#default-placeholders)

### 类似项目

[liquibase](https://www.liquibase.org/)

### 参考资料

[Flyway迁移指南最佳实践](https://www.cnblogs.com/vcmq/p/13194976.html)

[数据库版本管理工具Flyway——基础篇](https://blog.csdn.net/iteye_14867/article/details/82449872)

[Flyway 学习使用总结](https://blog.csdn.net/wbrg593/article/details/116517656)

[flyway使用教程](https://www.jianshu.com/p/3f762f6ef5c1)

[使用Flyway管理数据库更新](https://www.cnblogs.com/weijs/p/12930622.html)

[Spring Boot 集成 Flyway，数据库也能做版本控制，太牛逼了！](https://segmentfault.com/a/1190000040259397)

[Flyway数据库版本控制器理解与使用](https://blog.csdn.net/li521wang/article/details/89928930)
