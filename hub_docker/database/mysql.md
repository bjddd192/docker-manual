# mysql

## 官方镜像

[Hub官方](https://hub.docker.com/_/mysql/)

## 启动命令

```sh
docker run --name mysql -d -p 3309:3306 \
    -e 'MYSQL_ROOT_PASSWORD=blf1#root' \
    -v /home/docker/mysql/data:/var/lib/mysql/ \
    mysql:5.7.24
```
