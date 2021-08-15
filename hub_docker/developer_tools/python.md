# python

### 单独运行python脚本

```sh
docker run -it --rm \
-v "$PWD":/usr/src/myapp \
-w /usr/src/myapp \
harbor.bjds.belle.lan/library/python:3.8.10-buster \
pip install simplejson && python monitor_dag_status.py
```

### 参考资料

[docker下debian镜像开启ssh, 允许root用密码登录](https://blog.csdn.net/weixin_30859423/article/details/96719095)
