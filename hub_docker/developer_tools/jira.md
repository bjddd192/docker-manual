# jara

[最新版本](https://www.atlassian.com/software/jira/update)

[Hub版本-cptactionhank/atlassian-jira-software](https://hub.docker.com/r/cptactionhank/atlassian-jira-software)

[github-cptactionhank/atlassian-jira-software](https://hub.docker.com/r/cptactionhank/atlassian-jira-software)

[Hub版本](https://hub.docker.com/r/haxqer/jira)

[github](https://github.com/haxqer/jira)

[atlassian/jira-software](https://hub.docker.com/r/atlassian/jira-software)

## 配置数据库

```sql
CREATE DATABASE db_jira CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
GRANT ALL PRIVILEGES ON `db_jira`.* TO 'user_jira' @'%' IDENTIFIED BY 'FtvvZH5OcG';
FLUSH PRIVILEGES;
```

## 7.3.6 版本

```sh
# 正式启用后不要轻易销毁容器，一定要做好备份
# docker stop jira && docker rm jira

docker run --name jira -d -p 18080:8080 --restart=always \
  --hostname sz19f-scm-jira \
  -u root \
  hub.wonhigh.cn/tools/atlassian-jira-software:7.3.6

# 持久化
mkdir -p /data/docker_volumn/jira/var/atlassian
mkdir -p /data/docker_volumn/jira/opt/atlassian
docker cp jira:/var/atlassian/jira /data/docker_volumn/jira/var/atlassian
docker cp jira:/opt/atlassian/jira /data/docker_volumn/jira/opt/atlassian

# 正式启用后不要轻易销毁容器，一定要做好备份
# docker stop jira && docker rm jira

docker run --name jira -d -p 18080:8080 --restart=always \
  --hostname sz19f-scm-jira \
  --dns=10.0.43.19 \
  -u root \
  -e CATALINA_OPTS="-Xms2g -Xmx8g" \
  -v /data/docker_volumn/jira/var/atlassian/jira:/var/atlassian/jira \
  -v /data/docker_volumn/jira/opt/atlassian/jira:/opt/atlassian/jira \
  hub.wonhigh.cn/tools/atlassian-jira-software:7.3.6
  
# 修改 jvm 参数
# /data/docker_volumn/jira/opt/atlassian/jira/bin/setenv.sh

# 修改 /data/docker_volumn/jira/var/atlassian/jira/dbconfig.xml 文件中的连接串
# 增加 &amp;useSSL=false 参数
# 例如：<url>jdbc:mysql://10.0.30.129:3306/db_jira?useUnicode=true&amp;characterEncoding=UTF8&amp;useSSL=false&amp;sessionVariables=default_storage_engine=InnoDB</url>
# &amp; = & ，在HTML中的&用&amp; 来表示。
# 如这里配置错误，会导致启动后进入安装页面，给人持久化出错的错觉。

# BXWK-8D26-GNWD-AS0G
# AAABiA0ODAoPeNp9kU1PwkAQhu/9FZt40cM2LRREkiZKuzFVWgjFj4OXpQ66BrZ1dovy7+0XAS1ynMy88z7zzlmcSxJyJHaPWPbQvho6l8Tz56RjdSzjDQHke5plgOZYJCAVsFehRSpdFs3ZbDoLYmZE+XoBOFk+KEDlUtvwUql5oiO+BncwcJzLQbd//flpJuna+BDIzZZimmPyzhX4XINbWlOrS+2e0ZjOtxlU27xJGLKZF9yMdy32nQncHugcajs7AhZysWohxIAbwMB3R89P93Tgd/r0Nnry6U1s3dZ8GaaveaLNsqAqXeovjmAWC8UGXI05nBorWLgHUgPWo3G+UAmKrIptL/4/zyOpHwuguK0wkVwm/4RwgrH1gManCGUc+DGL6Nju2U6332/WtI5ikXtk7LhbrDmWyiVfKTAm+MalULw6OxiFhodQFX+fv6qZHgvEcrTzK5iKJUOhmof4sA/5rnAnceNOzkt2UsNfvAwJ2/BVXhnWzK23nsj8kOBQt99Z1z9kXiNzMCwCFDx/emLQdJ6oKiK8uYPJByZNFnlUAhR/VAzgViCxGbreTA4vQogadVWuDw==X02j3
```

## 8.7.1 版本

未能成功持久化与破解插件，只能作为学习使用。

```sh
docker stop jira && docker rm jira

docker run --name jira -d -p 8080:8080 --restart=always \
  --hostname sz19f-scm-jira \
  -e JVM_MINIMUM_MEMORY=1024m \
  -e JVM_MAXIMUM_MEMORY=2048m \
  -e SET_PERMISSIONS=false \
  atlassian/jira-software:8.7.1-jdk8
  
  -e ATL_JDBC_URL="jdbc:mysql://10.0.30.129:3306/db_jira?useUnicode=true&characterEncoding=utf-8&useSSL=false" \
  -e ATL_JDBC_USER=user_jira \
  -e ATL_JDBC_PASSWORD=FtvvZH5OcG \
  -e ATL_DB_DRIVER=com.mysql.jdbc.Driver \
  -e ATL_DB_TYPE=mysql \
  -e ATL_DB_SCHEMA_NAME=db_jira \
# 持久化
mkdir -p /data/docker_volumn/jira/var/atlassian
mkdir -p /data/docker_volumn/jira/opt/atlassian
docker cp jira:/var/atlassian/application-data/jira /data/docker_volumn/jira/var/atlassian
docker cp jira:/opt/atlassian/jira /data/docker_volumn/jira/opt/atlassian

docker stop jira && docker rm jira

docker run --name jira -d -p 8080:8080 --restart=always \
--hostname sz19f-scm-jira \
-e JVM_MINIMUM_MEMORY=1024m \
-e JVM_MAXIMUM_MEMORY=2048m \
-e SET_PERMISSIONS=false \
  -v /data/docker_volumn/jira/var/atlassian/jira:/var/atlassian/application-data/jira \
  -v /data/docker_volumn/jira/opt/atlassian/jira:/opt/atlassian/jira \
  hub.wonhigh.cn/tools/jira-software:8.7.1

mkdir -p /data/docker_volumn/jira/var/atlassian
mkdir -p /data/docker_volumn/jira/opt/atlassian
docker cp jira:/var/atlassian/jira /data/docker_volumn/jira/var/atlassian
docker cp jira:/opt/atlassian/jira /data/docker_volumn/jira/opt/atlassian

# 修改 /data/docker_volumn/jira/var/atlassian/jira/dbconfig.xml 文件中的连接串
# 增加 &useSSL=false 参数

docker stop jira && docker rm jira

docker run --name jira -d -p 8080:8080 --restart=always \
  -u root \
  -v /data/docker_volumn/jira/var/atlassian/jira:/var/atlassian/jira \
  -v /data/docker_volumn/jira/opt/atlassian/jira:/opt/atlassian/jira \
  hub.wonhigh.cn/tools/atlassian-jira-software:8.1.0

#BM2J-MGVD-BCEP-VPN7
#AAABhg0ODAoPeNp9kU9PwkAQxe/9FJt40cM2bWmVkDRR2o2poYVQ5ORlLQOugW2d3aJ+e/uPgBY5TmbevN+8uUpLSWKOxPaIZY1sa+TZJAgXxLEcy9gggHzLiwLQnIgMpAK2Elrk0mfJgs1n8yhlRlLuXgGn62cFqHxqG0EuNc90wnfgD4euezcc3N5/fJhZvjPeBXKzp5iVmL1xBSHX4NfW1BpQ2zM608V3Ac22YBrHbB5ED5NDi30VAr9PdC613QMBi7nY9hBSwD1gFPrj2Hmi8eMypOOAzehylty1fAXmqzLTZl1Qla/1J0cwq4ViD77GEi6NVSw8AKkB29G0fFUZiqKJ7Sj+P88zqZ8LoLqtMpFcZv+EcIGx94DOpwplEoUpS+jE9mx34Dndmt5RLPHPjJ13SzXHWrnmWwXGFDdcCsWbs6NxbAQITfH3+duWaVkh1qPOr2AalgKF6h4SwjHkp8qdpJ07ua7ZSQt/8zIibM+3ZWPYMvfeeiHzU4JT3XFnW/8Ad5ojfTAsAhRKrHLwCNQ+GcuPlTk6Ti4bRK55CwIUOYz+HFEAARboZylwMi0ZhCf9uW8=X02iu
```

## 异常处理

[You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true](https://blog.csdn.net/wsjzzcbq/article/details/81435779)

## 参考资料

[JIRA 8.6安装和破解](https://www.dqzboy.com/jira-8-6%E5%AE%89%E8%A3%85%E4%B8%8E%E7%A0%B4%E8%A7%A3)

[Jira插件安装](https://blog.csdn.net/shangyuanlang/article/details/80972462)

[使用docker搭建JIRA服务器，破解JIRA服务器、破解JIRA收费插件](https://blog.csdn.net/qq_40140473/article/details/95515828)

[烂泥：jira7.3/7.2安装、中文及破解(20170829更新)](https://www.ilanni.com/?p=12119)
