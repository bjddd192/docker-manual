# MongoDB

## 官方镜像

[Hub官方](https://hub.docker.com/_/mongo/)

## 启动命令

```sh
docker run -d --name mongo -p 30017:27017 --restart=always \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=123456 \
  -v /home/docker/mongo/data:/data/db \
  mongo:3.4.18
```
