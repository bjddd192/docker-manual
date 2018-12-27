# Gitlab

## 官方镜像

[Hub官方](https://hub.docker.com/r/beginor/gitlab-ce/)

## 启动命令

```sh
mkdir -p /data/gitlab/etc
mkdir -p /data/gitlab/log
mkdir -p /data/gitlab/data

docker run \
    --detach \
    --publish 8443:443 \
    --publish 8080:80 \
    --name gitlab \
    --restart unless-stopped \
    --volume /data/gitlab/etc:/etc/gitlab \
    --volume /data/gitlab/log:/var/log/gitlab \
    --volume /data/gitlab/data:/var/opt/gitlab \
    beginor/gitlab-ce:11.3.0-ce.0
```