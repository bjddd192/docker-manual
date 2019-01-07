# 容器管理

## 常用命令

```sh
# 查看容器 IP 地址
docker inspect --format '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' 容器ID/NAME

# 查看 volume
docker volume ls

# 查看具体的 volume 对应的真实地址
docker volume inspect VOLUME_NAME
```