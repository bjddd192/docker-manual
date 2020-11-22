# Airflow

airflow 是一个编排、调度和监控workflow的平台，由Airbnb开源，现在在Apache Software Foundation 孵化。airflow 将workflow编排为由tasks组成的DAGs(有向无环图)，调度器在一组workers上按照指定的依赖关系执行tasks。同时，airflow 提供了丰富的命令行工具和简单易用的用户界面以便用户查看和操作，并且airflow提供了监控和报警系统。

### 官方镜像

[官网](https://airflow.apache.org)

[官方文档](https://airflow.apache.org/docs/1.10.9/)

[apache/airflow github](https://github.com/apache/airflow)

[apache/airflow dockerhub](https://hub.docker.com/r/apache/airflow)

[puckel/docker-airflow github](https://github.com/puckel/docker-airflow)

[puckel/docker-airflow dockerhub](https://registry.hub.docker.com/r/puckel/docker-airflow)

[airflow for Kubernetes | KubeApps Hub](https://hub.kubeapps.com/charts/stable/airflow)

### 建立数据库

```sql
create database db_airflow character set utf8 collate utf8_unicode_ci;

grant all privileges on `db_airflow`.* to 'user_airflow'@'%' identified by 'scm_airflow';
flush privileges;
```

### 启动命令

```sh
docker run -d --name airflow --restart=always \
	hub.wonhigh.cn/third/docker-airflow:1.10.9

# 持久化数据
docker cp airflow:/usr/local/airflow/airflow.cfg /data/airflow/airflow.cfg

# 生成 fernet_key
docker exec -it airflow python -c "from cryptography.fernet import Fernet; FERNET_KEY = Fernet.generate_key().decode(); print(FERNET_KEY)"

docker stop airflow && docker rm airflow

docker run -d --name airflow --restart=always -p 8080:8080 \
	-e LOAD_EX=y \
	-e EXECUTOR=Local \
	-v /data/airflow/airflow.cfg:/usr/local/airflow/airflow.cfg \
	hub.wonhigh.cn/tools/airflow:1.10.9_201118.06 webserver

docker exec -it airflow python -c "from cryptography.fernet import Fernet; FERNET_KEY = Fernet.generate_key().decode(); print(FERNET_KEY)"
```

### Airflow基础命令

```sh
airflow -h

# 启用安全后设置管理员用户
airflow resetdb
airflow create_user --lastname user --firstname admin --username admin --email yang.lei@belle.com.cn --role Admin --password admin2#Air
airflow create_user --lastname user --firstname view --username view --email view_user@mail.com --role Viewer --password view2#Air
```

### 参考资料

[AirFlow简介](https://www.cnblogs.com/cord/p/9450910.html)

[airflow原理](https://www.cnblogs.com/hongfeng2019/p/11847400.html)

[airflow + CeleryExecutor 环境搭建](https://www.cnblogs.com/cord/p/9226608.html)

[使用Airflow实现ETL调度](https://www.cnblogs.com/yuuken/p/11270159.html)

[docker部署airflow](https://www.cnblogs.com/barneywill/p/10397260.html)

[Airflow docker版配置、部署](https://blog.csdn.net/Frederick_w/article/details/105706861)

[Airflow with Docker 容器部署](https://medium.com/@cchangleo/airflow-with-docker-%E5%AE%B9%E5%99%A8%E9%83%A8%E7%BD%B2-part2-8ddb83dc2d4a)

[airflow 使用心得，从环境到部署上线](https://blog.csdn.net/youzi_yun/article/details/90141362)

[airflow时区问题](https://blog.csdn.net/luanpeng825485697/article/details/105116369)

[AirFlow 常见问题](https://blog.csdn.net/gqj54082161/article/details/103756354/)
