# ftp

## 官方镜像

[cturra/ntp](https://hub.docker.com/r/cturra/ntp)

## github

[cturra/ntp](https://github.com/cturra/docker-ntp)

## 启动命令

```sh
docker stop ntp && docker rm -f ntp

docker run --name=ntp      \
    --restart=always       \
    --detach=true          \
    --publish=123:123/udp  \
    --cap-add=SYS_TIME     \
    -e NTP_SERVERS="time.cloudflare.com,ntp2.aliyun.com,ntp3.aliyun.com,ntp4.aliyun.com" \
    cturra/ntp

# 查看容器的ntp状态的详细信息
docker exec ntp chronyc tracking

# 查看对等列表以验证每个已配置的ntp源状态的方法
docker exec ntp chronyc sources

# 查看有关配置的每个ntp源收集的测量的统计信息：
docker exec ntp chronyc sourcestats

# 验证
lsof -i:123
date -s "2019-08-07 09:06:23"
/usr/sbin/ntpdate 10.0.43.32
```

## 参考资料

[CentOS 7.6安装配置Chrony同步系统时钟](https://blog.51cto.com/qiuyue/2344678)
