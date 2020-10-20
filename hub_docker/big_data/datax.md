# DataX

DataX 是一个异构数据源离线同步工具，致力于实现包括关系型数据库(MySQL、Oracle等)、HDFS、Hive、ODPS、HBase、FTP等各种异构数据源之间稳定高效的数据同步功能。

[alibaba/DataX](https://github.com/alibaba/DataX)

### 容器化

[idocking/docker-datax](https://github.com/idocking/docker-datax)

### 常用命令

```sh
# 通过命令查看配置模板
# python /datax/bin/datax.py -r {YOUR_READER} -w {YOUR_WRITER}
python /datax/bin/datax.py -r streamreader -w streamwriter
python /datax/bin/datax.py -r mysqlreader -w mysqlwriter

# 执行同步任务
python /datax/bin/datax.py /datax/json/test.json
```

### datax-web

[WeiYe-Jing/datax-web](https://github.com/WeiYe-Jing/datax-web)

```sh
# 单独启停模块
./bin/stop.sh  -m datax-executor
./bin/start.sh -m datax-executor 
```

### 参考资料

[DataX配置及使用](https://yq.aliyun.com/articles/216355)