# Apollo

Apollo（阿波罗）是一款可靠的分布式配置管理中心，诞生于携程框架研发部，能够集中化管理应用不同环境、不同集群的配置，配置修改后能够实时推送到应用端，并且具备规范的权限、流程治理等特性，适用于微服务配置管理场景。

[github](https://github.com/apolloconfig/apollo)

[dockerhub](https://hub.docker.com/u/apolloconfig)

[Apollo配置中心介绍](https://github.com/apolloconfig/apollo/wiki/Apollo%E9%85%8D%E7%BD%AE%E4%B8%AD%E5%BF%83%E4%BB%8B%E7%BB%8D)

### 部署步骤

1. 初始化数据库
[数据库脚本](https://github.com/apolloconfig/apollo/tree/v1.9.2/scripts/sql)
2. 初始化 config 库下 serverconfig 表的 eureka.service.url 参数
3. 部署 apollo-configservice
4. 部署 apollo-adminservice
5. 初始化 portal 库下 serverconfig 表的 apollo.portal.envs 参数
6. 部署 apollo-portal
7. 培训 ingress、nginx 对外提供服务
