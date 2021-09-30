# ftp

## 官方镜像

[bogem/ftp](https://hub.docker.com/r/bogem/ftp)

## github

[bogem/dockerfiles](https://hub.docker.com/r/bogem/ftp)

## 启动命令

```sh
docker stop ftp && docker rm -f ftp

docker run --name ftp -d -p 20:20 -p 21:21 -p 47400-47470:47400-47470 --restart=always \
  -e FTP_USER=scm \
  -e FTP_PASS=scm2Ftp \
  -e PASV_ADDRESS=10.0.43.24 \
  -v /data/docker_volumn/ftp/data:/home/vsftpd \
  bogem/ftp:latest

# 浏览器验证
ftp://scm:scm2Ftp@10.0.43.24/
```

## 客户端选择

[FileZilla](https://filezilla-project.org)

支持全平台。

## 参考资料

[深入挖掘vsftpd服务](https://blog.51cto.com/467754239/1440658)
