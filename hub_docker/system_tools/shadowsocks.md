# shadowsocks

## shadowsocks-privoxy

- shadowsocks client for socks5 proxy
- privoxy for http proxy

### 我的镜像

[bjddd192/shadowsocks-privoxy](https://cloud.docker.com/repository/docker/bjddd192/shadowsocks-privoxy)

### 启动命令

```sh
docker stop shadowsocks-privoxy && docker rm shadowsocks-privoxy

docker run -d --name shadowsocks-privoxy -p 8118:8118 --restart=always \
  -e SERVER_ADDR=x.x.x.x \
  -e SERVER_PORT=8087 \
  -e PASSWORD=dockerMan \
  bjddd192/shadowsocks-privoxy:2.9.1
```

### 验证代理

```sh
export all_proxy=http://127.0.0.1:8118
export ftp_proxy=http://127.0.0.1:8118
export http_proxy=http://127.0.0.1:8118
export https_proxy=http://127.0.0.1:8118
export no_proxy=localhost,172.17.0.0/16,192.168.0.0/16.,127.0.0.1,10.10.0.0/16
curl -I www.google.com
# 翻墙下载测试
wget -P /tmp --no-check-certificate https://s3.amazonaws.com/railsinstaller/Windows/railsinstaller-3.3.0.exe
```

### 取消使用代理

```sh
while read var; do unset $var; done < <(env | grep -i proxy | awk -F= '{print $1}')
```

### 参考资料

[bluebu/shadowsocks-privoxy](https://hub.docker.com/r/bluebu/shadowsocks-privoxy)

[shadowsocks-libev](https://hub.docker.com/r/shadowsocks/shadowsocks-libev)