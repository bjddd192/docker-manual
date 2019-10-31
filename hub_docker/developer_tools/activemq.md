# activemq

## 官方镜像

[Hub官方](https://hub.docker.com/r/rmohr/activemq)

## 启动命令

### 持久化数据(方式一)

```sh
docker stop activemq && docker rm activemq

docker run --name activemq -d -p 61616:61616 -p 8161:8161 --restart=always \
    -v /data/docker_volumn/activemq:/data/docker_volumn/activemq \
    rmohr/activemq:5.10.2
    
# 持久化数据
mkdir -p /data/docker_volumn/activemq
docker cp activemq:/opt/activemq/conf /data/docker_volumn/activemq/conf
docker cp activemq:/opt/activemq/data /data/docker_volumn/activemq/data
    
docker stop activemq && docker rm activemq

docker run --name activemq -d -p 61616:61616 -p 8161:8161 -u root --restart=always \
    -v /data/docker_volumn/activemq/conf:/opt/activemq/conf \
    -v /data/docker_volumn/activemq/data:/opt/activemq/data \
    rmohr/activemq:5.10.2
```

### 持久化数据(方式二)

```sh
docker stop activemq && docker rm activemq

docker run --user root --rm -it \
    -v /data/docker_volumn/activemq/conf:/mnt/conf \
    -v /data/docker_volumn/activemq/data:/mnt/data \
    rmohr/activemq:5.10.2 bash

# 持久化数据
chown activemq:activemq /mnt/conf
chown activemq:activemq /mnt/data
cp -a /opt/activemq/conf/* /mnt/conf/
cp -a /opt/activemq/data/* /mnt/data/
exit

docker stop activemq && docker rm activemq

docker run --name activemq -d -p 61616:61616 -p 8161:8161 --restart=always \
    -v /data/docker_volumn/activemq/conf:/opt/activemq/conf \
    -v /data/docker_volumn/activemq/data:/opt/activemq/data \
    rmohr/activemq:5.10.2
```
