# 容器管理

## 常用命令

```sh
# docker 空间占用总体分析
docker system df

# 输出空间占用细节
docker system df -v

# 输出容器的空间占用
docker ps -s

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

# 解决 docker 终端宽度、高度显示不正确
docker exec -it --env COLUMNS=`tput cols` --env LINES=`tput lines` your_container_name /bin/bash

# 重命名容器
docker rename youthful_minsky nginx

# 更新容器的配置
docker update [OPTIONS] CONTAINER [CONTAINER...]
# OPTIONS说明
# --restart：当容器退出时重新启动的策略
# --memory-swap：交换空间，“-1”允许无限交换.
# --memory , -m：内存限制

# 查看容器的 FS 中有变化文件信息
docker diff nginx

# 查看容器中活动进程
docker top nginx

# 查看容器的公开端口
docker port nginx

# 关闭所有正在运行的容器
docker kill $(docker ps -q)

# 移除所有停止的容器
docker rm $(docker ps -a -q)

# 根据状态移除
docker rm $(docker ps -q -f 'status=exited')

# 根据标签移除
docker rm $(docker ps -a | grep rabbitmq | awk '{print $1}')
docker rm $(docker ps -a | grep "46 hours ago")
```

## 参考资料

[Docker 1.13最实用命令行：终于可以愉快地打扫房间了](http://blog.shurenyun.com/shurenyun-docker-204/)

[docker stats命令](https://www.yiibai.com/docker/stats.html)
