# verdaccio

Verdaccio是一个简单的、零配置要求的本地私有 npm 仓库。

[官网](https://verdaccio.org/)

[官方文档](https://verdaccio.org/docs/docker/)

[verdaccio dockerhub](https://hub.docker.com/r/verdaccio/verdaccio)

[verdaccio github](https://github.com/verdaccio/verdaccio)

[verdaccio docker-examples](https://github.com/verdaccio/verdaccio/tree/master/docker-examples)

### 容器化部署

```sh
# docker stop verdaccio && docker rm verdaccio

docker run --name verdaccio -d -p 4873:4873 --restart=always \
verdaccio/verdaccio:5.2.0

# 持久化数据
docker cp verdaccio:/verdaccio /data/docker_volumn/
# 赋权
chown -R 10001:65533 /data/docker_volumn/verdaccio

docker stop verdaccio && docker rm verdaccio

docker run --name verdaccio -d -p 4873:4873 --restart=always \
-e VERDACCIO_USER_NAME=root \
-v /data/docker_volumn/verdaccio/conf:/verdaccio/conf \
-v /data/docker_volumn/verdaccio/storage:/verdaccio/storage \
-v /data/docker_volumn/verdaccio/plugins:/verdaccio/plugins \
verdaccio/verdaccio:5.2.0
```

### 配置文件

```yaml
#
# This is the config file used for the docker images.
# It allows all users to do anything, so don't use it on production systems.
#
# Do not configure host and port under `listen` in this file
# as it will be ignored when using docker.
# see https://verdaccio.org/docs/en/docker#docker-and-custom-port-configuration
#
# Look here for more config file examples:
# https://github.com/verdaccio/verdaccio/tree/master/conf
#

# path to a directory with all packages
storage: /verdaccio/storage/data
# path to a directory with plugins to include
plugins: /verdaccio/plugins

web:
  # WebUI is enabled as default, if you want disable it, just uncomment this line
  #enable: false
  title: Verdaccio
  # comment out to disable gravatar support
  # gravatar: false
  # by default packages are ordercer ascendant (asc|desc)
  # sort_packages: asc
  # darkMode: true
  # logo: http://somedomain/somelogo.png
  # favicon: http://somedomain/favicon.ico | /path/favicon.ico

# translate your registry, api i18n not available yet
# i18n:
# list of the available translations https://github.com/verdaccio/ui/tree/master/i18n/translations
#   web: en-US

auth:
  htpasswd:
    file: /verdaccio/storage/htpasswd
    # Maximum amount of users allowed to register, defaults to "+infinity".
    # You can set this to -1 to disable registration.
    # max_users: 1000

# a list of other known repositories we can talk to
uplinks:
  npmjs:
    # url: https://registry.npmjs.org/
    url: https://registry.npm.taobao.org/

packages:
  '@*/*':
    # scoped packages
    access: $all
    publish: $authenticated
    unpublish: $authenticated
    proxy: npmjs

  '**':
    # allow all users (including non-authenticated users) to read and
    # publish all packages
    #
    # you can specify usernames/groupnames (depending on your auth plugin)
    # and three keywords: "$all", "$anonymous", "$authenticated"
    access: $all

    # allow all known users to publish/publish packages
    # (anyone can register by default, remember?)
    publish: $authenticated
    unpublish: $authenticated

    # if package is not available locally, proxy requests to 'npmjs' registry
    proxy: npmjs

# You can specify HTTP/1.1 server keep alive timeout in seconds for incoming connections.
# A value of 0 makes the http server behave similarly to Node.js versions prior to 8.0.0, which did not have a keep-alive timeout.
# WORKAROUND: Through given configuration you can workaround following issue https://github.com/verdaccio/verdaccio/issues/301. Set to 0 in case 60 is not enough.
server:
  keepAliveTimeout: 60

middlewares:
  audit:
    enabled: true

# log settings
logs: { type: stdout, format: pretty, level: http }

#experiments:
#  # support for npm token command
#  token: false
#  # enable tarball URL redirect for hosting tarball with a different server, the tarball_url_redirect can be a template string
#  tarball_url_redirect: 'https://mycdn.com/verdaccio/${packageName}/${filename}'
#  # the tarball_url_redirect can be a function, takes packageName and filename and returns the url, when working with a js configuration file
#  tarball_url_redirect(packageName, filename) {
#    const signedUrl = // generate a signed url
#    return signedUrl;
#  }

# This affect the web and api (not developed yet)
#i18n:
#web: en-US
```

### 参考资料

[Verdaccio 使用 Docker 安装及迁移教程](https://blog.csdn.net/xionghnly/article/details/106617667)

[npm私服搭建—verdaccio方案及其最佳实践](https://www.jianshu.com/p/d32ce7e9d4d8)
