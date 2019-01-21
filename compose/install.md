# 安装使用

[官方网站](https://github.com/docker/compose/releases)

## Centos7

```sh
curl -L https://github.com/docker/compose/releases/download/1.21.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose

docker-compose --version

# 安装命令自动补齐
yum -y install bash-completion 

curl -L https://raw.githubusercontent.com/docker/compose/$(docker-compose version --short)/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose

su -
```