# DataX

DataX 是一个异构数据源离线同步工具，致力于实现包括关系型数据库(MySQL、Oracle等)、HDFS、Hive、ODPS、HBase、FTP等各种异构数据源之间稳定高效的数据同步功能。

[alibaba/DataX](https://github.com/alibaba/DataX)

[wgzhao/DataX](https://github.com/wgzhao/DataX)

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

#### 配置mongodb数据源

**最重要的就是 datax-web 中验证数据库 `admin` 需要配置在 `地址` 中，数据库写在 `数据库名称` 中。** 

### 参考资料

[DataX配置及使用](https://blog.csdn.net/DONGYUXIA15810857916/article/details/78095266)

[大数据学习——dataX工具部署和源码编译](https://pianshen.com/article/4714318566/)

[datax-web配置mongodb数据源](https://blog.csdn.net/Lonely_Devil/article/details/109646053)

[datax增量抽取mongoDB](https://blog.csdn.net/csdn_wr/article/details/113183478)

[DataX调优](https://www.cnblogs.com/hit-zb/p/10940849.html)

[db2插件下载](https://download.csdn.net/user/scdy0901/)

[DataX 通用RDBMSWriter](https://www.shulanxt.com/datawarehouse/datax/dataxrdbmswriter)

[datax参数设置_DataX Web数据增量同步配置说明](https://blog.csdn.net/weixin_39641463/article/details/113027797)
