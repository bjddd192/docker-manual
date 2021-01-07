# Sentry

Sentry 是一个开源的实时错误追踪系统，可以帮助开发者实时监控并修复异常问题。它主要专注于持续集成、提高效率并且提升用户体验。Sentry 分为服务端和客户端 SDK，前者可以直接使用它家提供的在线服务，也可以本地自行搭建；后者提供了对多种主流语言和框架的支持，包括 React、Angular、Node、Django、RoR、PHP、Laravel、Android、.NET、JAVA 等。同时它可提供了和其他流行服务集成的方案，例如 GitHub、GitLab、bitbuck、heroku、slack、Trello 等。

[官网](https://sentry.io/welcome/)

### 官方镜像

[getsentry/onpremise github](https://github.com/getsentry/onpremise)

### 容器化安装

```sh
cd /data
git clone https://github.com/getsentry/onpremise.git
cd onpremise/
# 查看 tag 版本
git tag
# 切换 tag 版本
git checkout 20.11.0
# 这时候 git 可能会提示你当前处于一个“detached HEAD" 状态。
# 因为 tag 相当于是一个快照，是不能更改它的代码的。
# 如果要在 tag 代码的基础上做修改，你需要一个分支： 
git checkout -b develop_20.11.1 20.11.1
# 查看当前分支
git branch -a

# 安装 docker-compose 1.24.1
curl -L https://get.daocloud.io/docker/compose/releases/download/1.24.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
# cp /usr/local/bin/docker-compose /root/local/bin/docker-compose

docker-compose --version

# 安装命令自动补齐
yum -y install bash-completion 

curl -L https://get.daocloud.io/docker/compose/$(docker-compose version --short)/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose

# 根据环境的实际情况修改 docker-compose.yml、install.sh 文件

# 初始化 sentry
./install.sh

# 启动 sentry
docker-compose up -d
```

### 钉钉告警插件

[FeSeason/sentry-10-dingding](https://github.com/FeSeason/sentry-10-dingding)

修改 requirement.txt 文件增加插件 sentry-10-dingding，然后重新 install.sh 即可。

钉钉告警的关键字配置为：sentry，才能收到告警信息哦。

### 参考资料

[Sentry 入门实战](http://sinhub.cn/2019/07/getting-started-guide-of-sentry/)

[自建sentry服务器后，无法收到邮件问题](https://blog.csdn.net/socct_yj/article/details/103039698)

[Sentry快速开始并集成钉钉群机器人](https://www.cnblogs.com/cjsblog/p/10585213.html)
