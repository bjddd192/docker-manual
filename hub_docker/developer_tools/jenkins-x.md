# jenkins-x

Jenkins X 是一个高度集成化的CI/CD平台，基于Jenkins和Kubernetes实现，旨在解决微服务体系架构下的云原生应用的持续交付的问题，简化整个云原生应用的开发、运行和部署过程。

[jenkins-x官网](https://jenkins-x.io/zh/)

[jenkins-x文档](https://jenkins-x.io/zh/docs/managing-jx/common-tasks/install-on-cluster/)

[Jenkins X cloud environments](https://github.com/qinyujia/cloud-environments)

### 前置条件

Kubernetes平台 版本 > 1.8
确认开启 RBAC
配置有默认的 Storage Class
helm
git私服

### 安装jenkins-x

[Setup Jenkins X on on-premises Kubernetes](https://jenkins-x.io/v3/admin/platforms/on-premises/)

#### 创建集群的Git仓库

基于[jx3-gitops-repositories/jx3-kubernetes](https://github.com/jx3-gitops-repositories/jx3-kubernetes)

#### 安装jx 3.x CLI

[jx download](https://github.com/jenkins-x/jx/releases/)

```sh
cd /tmp
curl -L "http://10.0.43.24:8066/package/jenkins-x/v3.2.151/jx-linux-amd64.tar.gz" | tar xzv "jx"
mv jx /usr/local/bin
jx version
```

#### 安装helm

```sh
cd /tmp
wget http://10.0.43.24:8066/helm/helm-v2.11.0-linux-amd64.tar.gz
tar -zxvf helm-v2.11.0-linux-amd64.tar.gz 
mv linux-amd64/helm /usr/local/bin/helm
helm help
helm version
# 添加自动补全
source <(helm completion bash)
```

#### 安装tiller

```sh
helm init --upgrade --tiller-image registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.11.0 \
--stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts

# 为 Tiller 设置帐号
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'

# 验证 Tiller 是否安装成功
kubectl get deploy --namespace kube-system tiller-deploy --output yaml|grep  serviceAccount
kubectl -n kube-system get pods|grep tiller
helm version
```

#### 安装chartmuseum

### 参考资料

[在云时代，如何选择一款合适的流水线工具？](https://time.geekbang.org/column/article/183907)

[Jenkins X 不是 Jenkins ，而是一个技术栈](https://www.chenshaowen.com/blog/jenkins-x-is-not-jenkins-but-stack.html)

[Jenkins X 实践系列](https://www.cnblogs.com/xiaoqi/p/jenkins-x-part1.html)

[Kubernetes持续交付-Jenkins X的Helm部署](https://developer.aliyun.com/article/680174)

[Jenkins X - Kubernetes指南](https://kubernetes.feisky.xyz/apps/devops/jenkinsx)

[DevOps 的打开方式: 构建和部署](https://jenkins-zh.cn/wechat/articles/2020/09/2020-09-02-devops-adoption-approach-build-and-deploy/)

[阿里云容器服务Kubernetes之Jenkins X（1）-安装部署实践篇](https://developer.aliyun.com/article/657149)

[阿里云容器服务Kubernetes之Jenkins X（2）-自动化CICD实践篇](https://developer.aliyun.com/article/657285)

[Terraform概述](https://www.alibabacloud.com/help/zh/doc-detail/91285.htm)

[Jenkins X v3对流水线提供了开箱即用的追踪支持](https://cloud.tencent.com/developer/article/1815855)
