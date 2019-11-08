# fastdfs

FastDFS是一个开源的轻量级分布式文件系统，它对文件进行管理，功能包括：文件存储、文件同步、文件访问（文件上传、文件下载）等，解决了大容量存储和负载均衡的问题。特别适合以文件为载体的在线服务，如相册网站、视频网站等等。

FastDFS为互联网量身定制，充分考虑了冗余备份、负载均衡、线性扩容等机制，并注重高可用、高性能等指标，使用FastDFS很容易搭建一套高性能的文件服务器集群提供文件上传、下载等服务。

## 官方镜像

[qbanxiaoli/fastdfs](https://hub.docker.com/r/qbanxiaoli/fastdfs)

## github

[qbanxiaoli/fastdfs](https://github.com/qbanxiaoli/fastdfs)

[happyfish100/fastdfs](https://github.com/happyfish100/fastdfs)

## 启动命令

```sh
docker stop fastdfs && docker rm -f fastdfs

docker run --name fastdfs -d --privileged=true --net=host --restart=always \
  -e IP=123.207.85.155 \
  -v /data/docker_volumn/fastdfs:/var/local/fdfs \
  qbanxiaoli/fastdfs

# 验证
docker exec -it fastdfs bash
echo "Hello FastDFS!" > index.html
fdfs_test /etc/fdfs/client.conf upload index.html
# 获取返回的文件地址，并在浏览器访问，如：
http://10.0.43.38:8080/group1/M00/00/00/CgArJlzOj7uAEBL_AAAAD1NdKGM56.html
http://10.0.43.38:8080/group1/M00/00/00/CgArJl3CQjeAWZw3AAAADwuG_C053_big.html
```

## 参考资料

[FastDFS 分布式文件系统](https://blog.csdn.net/kamroselee/article/details/80334621)

[FastDFS的介绍](https://www.cnblogs.com/shenxm/p/8459292.html)

[Nginx和FastDfs完整配置过程](https://blog.csdn.net/qq_34301871/article/details/80060235)

[FastDFS集群部署](https://www.cnblogs.com/cnmenglang/p/6731209.html)

[FastDFS Java Client](https://cloud.tencent.com/developer/article/1407660)

[FastDFS入门一篇就够](https://segmentfault.com/a/1190000018251300?utm_source=tag-newest)
