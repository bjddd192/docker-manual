# 容器管理

## 常用命令

```sh
# 查看容器 IP 地址
docker inspect --format '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' 容器ID/NAME

# 查看 volume
docker volume ls

# 查看具体的 volume 对应的真实地址
docker volume inspect VOLUME_NAME

# docker exec 以 root 身份登录容器
docker exec -it --user root <容器ID> /bin/bash

# 显示容器的实时流资源使用统计信息
docker stats
# 格式化显示所有容器的 CPU、内存信息
docker stats -a --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"
# 只拉取一次结果
docker stats -a --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}" --no-stream
```

## 参考资料

[Docker 1.13最实用命令行：终于可以愉快地打扫房间了](http://blog.shurenyun.com/shurenyun-docker-204/)

[docker stats命令](https://www.yiibai.com/docker/stats.html)
