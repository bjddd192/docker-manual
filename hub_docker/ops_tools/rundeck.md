# rundeck

## 官方镜像

[Hub官方](https://hub.docker.com/r/rundeck/rundeck)

## 启动命令

```sh
mkdir -p /data/docker_volumn/rundeck/server/config

docker stop rundeck && docker rm rundeck

docker run -d --name rundeck \
  rundeck/rundeck:3.0.11  

docker cp rundeck:/home/rundeck/server/config/realm.properties /data/docker_volumn/rundeck/server/config/

docker stop rundeck && docker rm rundeck

docker run -d --name rundeck -p 4440:4440 --restart always \
  -e RUNDECK_DATABASE_DRIVER=com.mysql.jdbc.Driver \
  -e RUNDECK_GRAILS_URL=http://10.0.43.24:4440 \
  -e RUNDECK_DATABASE_URL=jdbc:mysql://10.0.30.39:3306/db_rundeck?autoReconnect=true \
  -e RUNDECK_DATABASE_USERNAME=usr_rundeck \
  -e RUNDECK_DATABASE_PASSWORD=scm_rundeck \
  -v /data/docker_volumn/rundeck/server/data:/home/rundeck/server/data \
  -v /root/.ssh/:/home/rundeck/.ssh \
  -v /data/docker_volumn/rundeck/server/config/realm.properties:/home/rundeck/server/config/realm.properties \
  rundeck/rundeck:3.0.11
```

## 参考资料

[rundeck用户权限管理](https://blog.csdn.net/wangtingting_100/article/details/80914486)
