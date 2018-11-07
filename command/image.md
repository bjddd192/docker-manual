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
```