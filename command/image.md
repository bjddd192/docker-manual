# 镜像管理

## 常用命令

```sh
# 查找所有包含 registry.eyd.com:5000 关键字的镜像
docker images | grep registry.eyd.com:5000 | awk '{print $1":"$2}'

# 删除所有包含 registry.eyd.com:5000 关键字的镜像
docker rmi $(docker images | grep registry.eyd.com:5000 | awk '{print $1":"$2}')
docker images | grep registry.eyd.com:5000 | awk '{print $1":"$2}'

# 列举悬空镜像（TAG=none）
docker images -f dangling=true

# 删除全部悬空镜像
docker image prune

# 删除所有未被使用的镜像
docker image prune -a

# 查看 Docker 整体磁盘使用率的概况，包括镜像、容器和（本地）volume
docker system df 

# 删除当前没有被使用的一切项目，它按照一种正确的序列进行清理，所以会达到最大化的输出结果。
# 首先删除没有被使用的容器，然后是volume和网络，最后是挂起的镜像。
docker system prune
# 连同未使用的镜像一并清理
docker system prune -a
# 强制清理所有无用对象
docker system prune --volumes -a -f
```

## 参考资料

[Docker 1.13最实用命令行：终于可以愉快地打扫房间了](http://blog.shurenyun.com/shurenyun-docker-204/)