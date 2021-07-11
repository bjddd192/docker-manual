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

[Jenkins X on Kubernetes 兼容性对照表](https://jenkins-x.io/about/capabilities/)

### 安装jenkins-x

[Setup Jenkins X on on-premises Kubernetes](https://jenkins-x.io/v3/admin/platforms/on-premises/)

[Install the Operator](https://jenkins-x.io/v3/admin/setup/operator/)

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
# wget https://get.helm.sh/helm-v3.5.4-linux-amd64.tar.gz
wget http://10.0.43.24:8066/helm/helm-v3.5.4-linux-amd64.tar.gz
tar -zxvf helm-v3.5.4-linux-amd64.tar.gz 
mv linux-amd64/helm /usr/local/bin/helm
helm help
helm version
# 添加自动补全
source <(helm completion bash)
```

#### 安装tiller

忽略，新版本helm已不需要安装tiller。

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

#### 安装git operator

```sh
# 安装之前设置代理解决需翻墙下载镜像问题
# gitlab.wonhigh.cn是内网的地址，改为IP，不然安装jx-boot会卡住
# jx admin operator --url=http://gitlab.wonhigh.cn/scm-ops/jx3-kubernetes.git --username scm-readonly --token S5bpgWgC_tmxzAyZz6z4

# kubectl delete ns jx jx-git-operator jx-production jx-staging kuberhealthy nginx tekton-pipelines

export HPROXY=""
export NPROXY=""

export HPROXY=http://10.234.6.219:8118
export NPROXY=*.svc\\,*.belle.net.cn\\,*.wonhigh.cn\\,*.lesoon.com\\,localhost\\,10.0.30.203\\,172.17.191.26\\,127.0.0.1\\,.local\\,0\\,1\\,2\\,3\\,4\\,5\\,6\\,7\\,8\\,9

jx admin operator --url=http://10.0.30.203/root/jx3-kubernetes.git \
--username root --token tEsEWSsf6xzMJZHkKNUe \
--set jxBootJobEnvVarSecrets.HTTP_PROXY=$HPROXY \
--set jxBootJobEnvVarSecrets.HTTPS_PROXY=$HPROXY \
--set jxBootJobEnvVarSecrets.http_proxy=$HPROXY \
--set jxBootJobEnvVarSecrets.https_proxy=$HPROXY \
--set jxBootJobEnvVarSecrets.NO_PROXY=$NPROXY \
--set jxBootJobEnvVarSecrets.no_proxy=$NPROXY

jx admin operator --url=http://172.17.191.26/scm-ops/jx3-kubernetes.git \
--username "scm-readonly" --token S5bpgWgC_tmxzAyZz6z4 \
--set jxBootJobEnvVarSecrets.HTTP_PROXY=$HPROXY \
--set jxBootJobEnvVarSecrets.HTTPS_PROXY=$HPROXY \
--set jxBootJobEnvVarSecrets.http_proxy=$HPROXY \
--set jxBootJobEnvVarSecrets.https_proxy=$HPROXY \
--set jxBootJobEnvVarSecrets.NO_PROXY=$NPROXY \
--set jxBootJobEnvVarSecrets.no_proxy=$NPROXY

# jx admin operator --url=https://github.com/bjddd192/jx3-kubernetes.git \
# --username "bjddd192" --token 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 \
# --set jxBootJobEnvVarSecrets.HTTP_PROXY=$HPROXY \
# --set jxBootJobEnvVarSecrets.HTTPS_PROXY=$HPROXY \
# --set jxBootJobEnvVarSecrets.http_proxy=$HPROXY \
# --set jxBootJobEnvVarSecrets.https_proxy=$HPROXY \
# --set jxBootJobEnvVarSecrets.NO_PROXY=$NPROXY \
# --set jxBootJobEnvVarSecrets.no_proxy=$NPROXY

# 健康检查
jx health status -A

jx admin operator --username mygituser --token mygittoken
```

#### 相关镜像

```sh
docker pull hub.wonhigh.cn/k8s/nfs-client-provisioner:latest
docker pull busybox:1.28.4

docker pull gcr.io/jenkinsxio/jx-git-operator:0.0.177
docker pull ghcr.io/jenkins-x/jx-boot:3.2.151

kind load docker-image gcr.io/jenkinsxio/jx-git-operator:0.0.177 gcr.io/jenkinsxio/jx-git-operator:0.0.177
kind load docker-image ghcr.io/jenkins-x/jx-boot:3.2.151 ghcr.io/jenkins-x/jx-boot:3.2.151

kind load docker-image hub.wonhigh.cn/k8s/nfs-client-provisioner:latest hub.wonhigh.cn/k8s/nfs-client-provisioner:latest
kind load docker-image busybox:1.28.4 busybox:1.28.4
```

### 解码 Secret

```sh
kubectl -n jx get secret tekton-git -o jsonpath='{.data}' 
echo "c2NtLXJlYWRvbmx5" | base64 --decode
```

### 异常处理

Q：Error: .jx/secret/mapping/secret-mappings.yaml file not found
A：[Cannot install jx operator in existing kubernetes cluster](https://gitmemory.com/issue/jenkins-x/jx/7679/821779805)

### 参考资料

[在云时代，如何选择一款合适的流水线工具？](https://time.geekbang.org/column/article/183907)

[迈向云端：云原生应用时代的平台思考](https://time.geekbang.org/column/article/180496)

[Jenkins X 不是 Jenkins ，而是一个技术栈](https://www.chenshaowen.com/blog/jenkins-x-is-not-jenkins-but-stack.html)

[Jenkins X 实践系列](https://www.cnblogs.com/xiaoqi/p/jenkins-x-part1.html)

[Kubernetes持续交付-Jenkins X的Helm部署](https://developer.aliyun.com/article/680174)

[Jenkins X - Kubernetes指南](https://kubernetes.feisky.xyz/apps/devops/jenkinsx)

[DevOps 的打开方式: 构建和部署](https://jenkins-zh.cn/wechat/articles/2020/09/2020-09-02-devops-adoption-approach-build-and-deploy/)

[阿里云容器服务Kubernetes之Jenkins X（1）-安装部署实践篇](https://developer.aliyun.com/article/657149)

[阿里云容器服务Kubernetes之Jenkins X（2）-自动化CICD实践篇](https://developer.aliyun.com/article/657285)

[Terraform概述](https://www.alibabacloud.com/help/zh/doc-detail/91285.htm)

[Jenkins X v3对流水线提供了开箱即用的追踪支持](https://cloud.tencent.com/developer/article/1815855)

[Running Jenkins-X locally on Kind](https://medium.com/@dantwining_26268/running-jenkins-x-locally-on-kind-4b43be1de418)
