# rundeck

[官方网站](https://www.rundeck.com/open-source)

[github](https://github.com/rundeck/rundeck)

[版本变更信息](https://github.com/rundeck/rundeck/blob/master/CHANGELOG.md)

[官方文档](https://docs.rundeck.com/docs/)

[官方安装教程](https://docs.rundeck.com/docs/administration/install/docker.html)

## 官方镜像

[Hub官方](https://hub.docker.com/r/rundeck/rundeck)

## 安装步骤

### 准备数据库

```sql
create database db_rundeck;
grant all privileges on db_rundeck.* to 'usr_rundeck'@'%' identified by 'scm_rundeck';
flush privileges;
```

### 编排文件(非持久化启动)

```yaml
version: '2.1'
services:
  rundeck:
    image: rundeck/rundeck:3.0.20
    container_name: rundeck
    restart: always
    ports:
    - "4440:4440"
    environment:
    - RUNDECK_DATABASE_DRIVER=com.mysql.jdbc.Driver
    - RUNDECK_GRAILS_URL=http://10.243.2.164:4440
    - RUNDECK_DATABASE_URL=jdbc:mysql://10.243.2.164:3306/db_rundeck?autoReconnect=true
    - RUNDECK_DATABASE_USERNAME=usr_rundeck
    - RUNDECK_DATABASE_PASSWORD=scm_rundeck
    - RUNDECK_JAAS_MODULES_0=ReloadablePropertyFileLoginModule
    - RUNDECK_JAAS_MODULES_1=PropertyFileLoginModule
    volumes:
    - /etc/localtime:/etc/localtime
    network_mode: bridge
    user: root
```

### 初始化配置文件夹

```sh
# 创建 rundeck 容器
docker-compose up -d rundeck

# 从容器中复制需持久化的文件夹
mkdir -p /data/docker_volumn/rundeck/server
docker cp rundeck:home/rundeck/server/data /data/docker_volumn/rundeck/server/
docker cp rundeck:/home/rundeck/.ssh /data/docker_volumn/rundeck/server/
docker cp rundeck:home/rundeck/server/config /data/docker_volumn/rundeck/server/
docker cp rundeck:/home/rundeck/etc /data/docker_volumn/rundeck/server/
mv /data/docker_volumn/rundeck/server/.ssh /data/docker_volumn/rundeck/server/ssh

# 销毁 rundeck 容器
docker-compose stop rundeck
docker-compose rm -f rundeck
```

### 编排文件(持久化启动)

```yaml
version: '2.1'
services:
  rundeck:
    image: rundeck/rundeck:3.0.20
    container_name: rundeck
    restart: always
    ports:
    - "4440:4440"
    environment:
    - RUNDECK_DATABASE_DRIVER=com.mysql.jdbc.Driver
    - RUNDECK_GRAILS_URL=http://10.243.2.164:4440
    - RUNDECK_DATABASE_URL=jdbc:mysql://10.243.2.164:3306/db_rundeck?autoReconnect=true
    - RUNDECK_DATABASE_USERNAME=usr_rundeck
    - RUNDECK_DATABASE_PASSWORD=scm_rundeck
    - RUNDECK_JAAS_MODULES_0=ReloadablePropertyFileLoginModule
    - RUNDECK_JAAS_MODULES_1=PropertyFileLoginModule
    volumes:
    - /etc/localtime:/etc/localtime
    - /data/docker_volumn/rundeck/server/data:/home/rundeck/server/data
    - /data/docker_volumn/rundeck/server/ssh:/home/rundeck/.ssh
    - /data/docker_volumn/rundeck/server/config:/home/rundeck/server/config
    - /data/docker_volumn/rundeck/server/etc:/home/rundeck/etc
    network_mode: bridge
    user: root
```

### 验证

访问：http://10.243.2.164:4440，初始密码：admin/admin

## 节点配置

### 配置文件

容器内文件路径：`/home/rundeck/server/data/resources.xml`

宿主机文件路径：`/data/docker_volumn/rundeck/server/data/resources.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project>
    <node name="10.243.2.165"
        description="k8s培训集群主控节点" tags="k8s"
        osArch="amd64" osFamily="unix" osName="Linux"
        osVersion="3.10.0-327.el7.x86_64"
        hostname="10.243.2.165:22"  username="root"
    />
</project>
```

### 设置免密

首先拷贝宿州机器的ssh

```sh
docker exec -it --env COLUMNS=200 --env LINES=200 rundeck bash
ssh-copy-id -i /home/rundeck/.ssh/id_rsa.pub -p 22 root@10.243.2.165
scp -r -P 22 -i /home/rundeck/.ssh/id_rsa 10.243.2.165:/tmp/nothing . 
```

## 用户管理

### 加密密码

用户文件位于：`/data/docker_volumn/rundeck/server/config/realm.properties`

```sh
docker exec -it rundeck java -jar rundeck.war --encryptpwd Jetty 
$ java -jar rundeck-3.0.0.war --encryptpwd Jetty
Required values are marked with: *
Username (Optional, but necessary for Crypt encoding):
jsmith    <-----Type this value
*Value To Encrypt (The text you want to encrypt):
mypass    <-----Type this value

obfuscate: OBF:1xfd1zt11uha1ugg1zsp1xfp
md5: MD5:a029d0df84eb5549c641e04a9ef389e5
crypt: CRYPT:jsnDAc2Xk4W4o
```

### 添加用户

编辑 `realm.properties` 文件，使用如下类似的行将其添加到文件中：

```properties
admin:admin2Leo,user,admin
deployer:scm_uat_ops,user
jsmith:MD5:a029d0df84eb5549c641e04a9ef389e5,user
```

关于用户配置文件修改后需要热加载，请查看官方文档：

[验证用户](https://docs.rundeck.com/docs/administration/security/authenticating-users.html#container-authentication-and-authorization)

应该是调整 `/data/docker_volumn/rundeck/server/config/jaas-loginmodule.conf` 文件配置即可。

目前我使用的是重启容器生效的策略，简单粗暴。

### 配置普通用户权限

在访问策略中增加 ACL Policy：

```yaml
description: user access projects control
context:
  project: '.*' # all projects
for:
  job:
    - allow: [read,run,kill] # allow read/write/delete/run/kill of all jobs
  node:
    - allow: [read,run] # allow read/run for all nodes
by:
  group: user

---

description: user access rundeck control
context:
  application: 'rundeck'
for:
  project:
    - allow: 'read' # allow view/admin of all projects
  storage:
    - allow: 'read' # allow read/create/update/delete for all /keys/* storage content
by:
  group: user
```

## 下拉框调用远程API

```sh
# 格式如下：
http://10.0.43.37:8000/api/k8s_app_deploy_config/?app_env=dev#timeout=60;contimeout=5;retry=3
http://10.0.43.37:8000/api/k8s_app_deploy_config/?app_env=dev&app_namespace=${option.app_ns.value}#timeout=60;contimeout=5;retry=3
```

## 参考资料

[rundeck用户权限配置](https://blog.51cto.com/haoyonghui/2086774)

[rundeck权限设置](http://www.manongjc.com/article/64995.html)

[修改docker -v 挂载的文件遇到的问题](https://lrita.github.io/2017/08/18/mount-volume-to-docker-and-modify-from-host/)
