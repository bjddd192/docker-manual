# spug

Jumpserver 是全球首款完全开源的堡垒机，使用 GNU GPL v2.0 开源协议，是符合 4A 的专业运维审计系统。

[github](https://github.com/openspug/spug)

[官方文档](https://spug.cc/docs/install-docker)

### Docker 快速安装

```sh
docker run -d --restart=always --name=spug -p 80:80 --restart=always \
-v /data/docker_volumn/spug:/data \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /usr/local/bin/docker:/usr/bin/docker \
registry.aliyuncs.com/openspug/spug

# 初始化账号密码
docker exec spug init_spug admin spug.dev
docker restart spug

# http://10.10.234.80/
# 用户名： admin  
# 密码： spug.dev
```

## 参考资料
