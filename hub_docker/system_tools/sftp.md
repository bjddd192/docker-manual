# sftp

## 官方镜像

[atmoz/sftp](https://hub.docker.com/r/atmoz/sftp)

## github

[atmoz/sftp](https://www.github.com/atmoz/sftp)

## 启动命令

```sh
echo -n "scm2Ftp" | docker run -i --rm atmoz/makepasswd:latest --crypt-md5 --clearfrom=- 
# 获得加密的密码：scm2Ftp   $1$pHK5AlwG$phlh2vFjyawxmc/YNwUvs1

# 生成用户密钥文件
mkdir -p /data/docker_volumn/sftp/config
echo 'scm:$1$pHK5AlwG$phlh2vFjyawxmc/YNwUvs1:e:1001:1001' > /data/docker_volumn/sftp/config/users.conf

# 创建数据目录
mkdir -p /data/docker_volumn/sftp/data
chown -R 1001:1001 /data/docker_volumn/sftp/data

docker stop sftp && docker rm -f sftp

docker run --name sftp -d -p 11022:22 --restart=always \
  -v /data/docker_volumn/sftp/data:/home/scm/sftp/ \
  -v /data/docker_volumn/sftp/config/users.conf:/etc/sftp/users.conf:ro \
  atmoz/sftp:latest

# 验证
sftp -P 11022 scm@127.0.0.1:/sftp
```

## 客户端选择

[FileZilla](https://filezilla-project.org)

支持全平台。
