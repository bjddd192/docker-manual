# devpi

devpi包含三个组件:

- devpi-server: 提供镜像与缓存功能，在企业内网部署，提高下载python package的效率
- devpi-web: 提供web界面与查询功能
- devpi-client: 命令行工具, 提供包上传等功能

[muccg/devpi hub_docker](https://hub.docker.com/r/muccg/devpi)

[muccg/docker-devpi github](https://github.com/muccg/docker-devpi)

### 启动命令

```sh
# DEVPI_SERVER_VERSION=4.5.0
# DEVPI_WEB_VERSION=3.3.0
# DEVPI_CLIENT_VERSION=4.0.2

# 为了增加安全性，--restrict-modify root已添加该参数，因此只有 root 可以创建用户和索引。

# docker stop devpi && docker rm devpi

docker run --name devpi -d -p 3141:3141 --restart=always \
-e DEVPI_PASSWORD=123456 \
-e PIP_INDEX_URL=https://mirrors.aliyun.com/pypi/simple \
muccg/devpi:4.5.0


docker exec -it devpi bash
devpi use http://localhost:3141
devpi login root --password=123456
devpi use dev
devpi index -c dev
# 查看索引信息
devpi getjson /root
# 修改默认镜像源地址(devpi默认使用的官方镜像源地址，慢且不稳定，修改成豆瓣源)
devpi index pypi type=mirror mirror_url=https://mirrors.aliyun.com/pypi/simple mirror_web_url_fmt=https://mirrors.aliyun.com/pypi/simple/{name}/
```

### 验证私服

```sh
# web验证
http://10.0.43.13:3141/

# 上传 wheelhouse
# 随便找一个requirements.txt文件
docker run -it --rm hub.wonhigh.cn/library/python:3.7.4-alpine3.10 sh
pip wheel --wheel-dir /tmp/wheelhouse -r requirements.txt
pip install --no-cache-dir -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com devpi-client==4.0.2
devpi use http://10.0.43.13:3141/root/public
devpi login root
devpi upload --from-dir /tmp/wheelhouse

# 上传私有化包
docker run -it --rm hub.wonhigh.cn/library/python:3.7.4-buster bash
pip install --no-cache-dir -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com devpi-client==4.0.2
pip install --no-cache-dir -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com twine
cd /tmp/
mkdir -p /tmp/linode_example/linode_example/
tee /tmp/linode_example/linode_example/__init__.py <<-'EOF'
def hello_word():
     print("hello world")
EOF
tee /tmp/linode_example/setup.py <<-'EOF'
from setuptools import setup
setup(
     name='linode_example',
     packages=['linode_example'],    
     description='Hello world enterprise edition',
     version='0.1',   #版本号
     url='http://github.com/example/linode_example',
     author='Linode',
     keywords=['pip','linode','example']
     )
EOF
cd /tmp/linode_example
python3 setup.py sdist
devpi use http://10.0.43.13:3141/root/public
devpi login root
devpi use root/public
devpi upload

# 查看索引
devpi list
```

### 问题处理

[devpi-server can't modify root/pypi index offline](https://github.com/devpi/devpi/issues/615)

升级 pip install devpi-server==4.8.1 版本解决

```sh
docker exec -it  devpi  bash
pip install devpi-server==4.8.1
devpi index pypi type=mirror mirror_url=https://mirrors.aliyun.com/pypi/simple mirror_web_url_fmt=https://mirrors.aliyun.com/pypi/simple/{name}/

# 重启 devpi
docker restart devpi
```

### 参考资料

[devpi: PyPI server and packaging/testing/release tool](https://devpi.net/docs/devpi/devpi/stable/%2Bd/index.html)

[使用 docker + devpi 搭建本地 pypi 源](https://segmentfault.com/a/1190000018781421)

[基于docker部署python私有仓库-devpi](https://blog.csdn.net/Stephen_Curry11/article/details/107566830)

[如何使用 devpi 搭建 PyPI Server](https://www.chenshaowen.com/blog/how-to-build-a-pypi-server-using-devpi.html)

[devpi搭建pip源服务器](https://www.bianchengquan.com/article/602635.html)
