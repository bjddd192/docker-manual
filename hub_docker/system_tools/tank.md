# tank

蓝眼云盘

## 官方镜像

[eyeblue/tank](https://hub.docker.com/r/eyeblue/tank)

## github

[eyeblue/tank](https://github.com/eyebluecn/tank)

## 启动命令

```sh
docker stop tank && docker rm -f tank

docker run --name tank -d -p 6010:6010 --restart=always \
  -v /data/docker_volumn/tank:/data/build/matter \
  eyeblue/tank:3.0.5

# 验证，在浏览器访问，如：
http://10.0.43.17:6010
```

## 参考资料