# 编排文件参考

[官方参考](https://docs.docker.com/compose/compose-file/)

## volumes

docker-compose里两种设置方式都是可以持久化的：

1. 绝对路径的

```yaml
ghost:
  image: ghost
  volumes:
    - ./ghost/config.js:/var/lib/ghost/config.js
```

2. 卷标的

```yaml
services:
  mysql:  
    image: mysql
    container_name: mysql
    volumes:
      - mysql:/var/lib/mysql
...
volumes:
  mysql:
```

第一种情况路径直接挂载到本地，比较直观，但需要管理本地的路径，而第二种使用卷标的方式，比较简洁，但你不知道数据存在本地什么位置，下面说明如何查看docker的卷标：

```sh
# 查看所有卷标
docker volume ls

# 查看具体的 volume 对应的真实地址
docker volume inspect redissentinel_redis_node1_data
```