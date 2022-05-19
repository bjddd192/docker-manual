# mysql-schema-sync

MySQL Schema 自动同步工具。

[github](https://github.com/hidu/mysql-schema-sync)

### 操作命令

```sh
# 预览并生成变更sql
mysql-schema-sync -conf config.json 2>/dev/null >db_alter.sql

# 直接运行同步
mysql-schema-sync -conf /go/bin/config.json -drop -sync

# docker运行
docker run -it --rm -v /tmp/config.json:/go/bin/config.json  harbor.bjds.belle.lan/tools/mysql-schema-sync:latest mysql-schema-sync -conf /go/bin/config.json -drop -sync
```

### 参考资料

[MySQL表结构自动同步工具mysql-schema-sync安装使用](https://www.cnblogs.com/DBABlog/p/12926944.html)

[推荐一款 MySQL 表结构自动同步工具 mysql-schema-sync](https://www.hi-linux.com/posts/4769.html)
