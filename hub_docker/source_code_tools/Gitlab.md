# Gitlab

### 官方镜像

[Hub官方](https://hub.docker.com/r/beginor/gitlab-ce/)

[gitlab/gitlab-ce](https://hub.docker.com/r/gitlab/gitlab-ce)

### 启动命令

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

### 参考资料

[Docker中安装了gitlab，忘记了管理员密码，进行管理员密码重置](https://www.cnblogs.com/zhang-yawei/p/12692493.html)

```sh
docker exec -it gitlab bash
> gitlab-rails console -e production
> user = User.where(id: 1).first
> user.password = 'secret_pass'
> user.password_confirmation = 'secret_pass'
> user.save!
> exit
```
