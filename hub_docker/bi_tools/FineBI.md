# FineBI

FineBI 是新一代自助大数据分析的 BI 软件。

[官网](https://www.finebi.com/)

[Hub官方](https://hub.docker.com/r/haodong/finebi)

[Github官方](https://github.com/haodong/docker-finebi)

FineBI 激活码：	ca37747f-71501c812-d34c-e9871871a22a

## 安装包下载

```sh
wget https://fine-build.oss-cn-shanghai.aliyuncs.com/finebi/5.1/public/exe/spider/linux_unix_FineBI5_1-CN.sh
```

## 容器化安装

```sh
docker stop finebi && docker rm finebi

docker run -d --name finebi -p 31199:37799 --restart=always \
  haodong/finebi:version-5.1-2019.1.15

# 持久化数据
docker cp finebi:/usr/local/FineBI5.1 /data/docker_volumn/FineBI5.1

docker stop finebi && docker rm finebi

docker run -d --name finebi -p 31199:37799 --restart=always \
  -v /data/docker_volumn/FineBI5.1:/usr/local/FineBI5.1 \
  haodong/finebi:version-5.1-2019.1.15
```

镜像比较大，启动比较慢，需要耐心等待。

测试地址：

http://10.0.43.13:31199/webroot/decision
