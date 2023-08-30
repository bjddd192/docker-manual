# jumpserver

Jumpserver 是全球首款完全开源的堡垒机，使用 GNU GPL v2.0 开源协议，是符合 4A 的专业运维审计系统。

[github](https://github.com/jumpserver/jumpserver)

[官方文档](http://docs.jumpserver.org/zh/docs/step_by_step.html)

## 官方镜像

[Hub官方](https://hub.docker.com/r/jumpserver/jms_all)

## Docker 快速安装

```sh
# 生成随机加密秘钥, 勿外泄
$ if [ "$SECRET_KEY" = "" ]; then SECRET_KEY=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 50`; echo "SECRET_KEY=$SECRET_KEY" >> ~/.bashrc; echo $SECRET_KEY; else echo $SECRET_KEY; fi

$ if [ "$BOOTSTRAP_TOKEN" = "" ]; then BOOTSTRAP_TOKEN=`cat /dev/urandom | tr -dc A-Za-z0-9 | head -c 16`; echo "BOOTSTRAP_TOKEN=$BOOTSTRAP_TOKEN" >> ~/.bashrc; echo $BOOTSTRAP_TOKEN; else echo $BOOTSTRAP_TOKEN; fi

# 以 docker-compose 编排的方式启动
```

```sql
create database jumpserver;
```

## 参考资料

[使用JumpServer管理你的服务器](http://www.imooc.com/article/285466)

## 3.0版本

[离线安装](https://docs.jumpserver.org/zh/v3/installation/setup_linux_standalone/offline_install/)

[MFA 认证](https://docs.jumpserver.org/zh/master/admin-guide/authentication/mfa/#1-mfa)
