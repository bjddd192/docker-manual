# sonarqube

代码审查工具。

从SonarQube 7.9开始，支持的数据库变为：Oracle、Microsoft SQL Server和PostgreSQL。Oracle和SQL Server是收费的商业数据库而PostgreSQL则是开源免费的。SonarQube聚焦于提供资源对PostgreSQL进行支持，以保证用户在使用免费数据库的基础上也能够获得同等的良好体验。

## 官方镜像

[官方文档](https://docs.sonarqube.org/latest/setup/install-server/)

[Hub官方](https://hub.docker.com/_/sonarqube)

## 创建sonar数据库

```sql
create database db_sonar;
grant all privileges on `db_sonar`.* to 'user_sonar'@'%' identified by 'scm2Sonar';
flush privileges;
```

## 启动命令

```sh
# 因内嵌了 Elasticsearch，需要添加内核参数
/sbin/sysctl -w vm.max_map_count=262144
# /sbin/sysctl -w fs.file-max=65536
# ulimit -n 65536
# ulimit -u 4096

docker stop sonarqube && docker rm sonarqube

docker run -d --name sonarqube \
    -p 9000:9000 \
    -e SONARQUBE_JDBC_URL="jdbc:postgresql://172.17.209.202:5432/db_sonar" \
    -e SONARQUBE_JDBC_USERNAME="root" \
    -e SONARQUBE_JDBC_PASSWORD="Scm2Postgres" \
    sonarqube:7.9.3-community
    
# 持久化数据
docker cp sonarqube:/opt/sonarqube /data/docker_volumn/
chmod 777 -R /data/docker_volumn/sonarqube/

docker stop sonarqube && docker rm sonarqube

docker run -d --name sonarqube -p 9000:9000 --restart=always \
    -e SONARQUBE_JDBC_URL="jdbc:postgresql://172.17.209.202:5432/db_sonar" \
    -e SONARQUBE_JDBC_USERNAME="root" \
    -e SONARQUBE_JDBC_PASSWORD="Scm2Postgres" \
    -v /data/docker_volumn/sonarqube:/opt/sonarqube \
    sonarqube:7.9.3-community
```

汉化：

应用市场搜索 Chinese Pack 并安装。

## 验证

访问 url 如：http://172.17.209.202:9000

## 参考资料

[docker + mysql 安装 sonarqube](https://www.cnblogs.com/freestudy/p/10281744.html)

[Docker创建Sonar容器，数据库使用Mysql5.7](https://www.pianshen.com/article/802721241/)

[Sonar代理设置](https://blog.csdn.net/youlinmin/article/details/53896079)
