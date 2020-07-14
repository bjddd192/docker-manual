# pypiserver

部署私有 pip 库需要使用 pypiserver 这个 Python 包，方便快捷。

## 官方镜像

[gitlab](https://github.com/pypiserver/pypiserver/blob/master/docker-compose.yml)

[Hub官方](https://hub.docker.com/r/pypiserver/pypiserver)

## 启动命令

```sh
mkdir -p /data/docker_volumn/packages
mkdir -p /data/docker_volumn/auth

# 生成 htpass 文件
docker stop htpasswd && docker rm htpasswd

docker run --name htpasswd -it --rm \
  -u root \
  -v /data/docker_volumn/auth:/data/auth \
  xmartlabs/htpasswd scm_python Dev2python > /data/docker_volumn/auth/.htaccess

docker stop pypiserver && docker rm pypiserver

docker run --name pypiserver -d -p 7080:8080 --restart=always \
  -u root \
  -v /data/docker_volumn/packages:/data/packages \
  -v /data/docker_volumn/auth:/data/auth \
  pypiserver/pypiserver:v1.3.2 \
  -P /data/auth/.htaccess -a update,download,list /data/packages

```

## 验证

访问 url 如：http://172.17.209.53:7080
