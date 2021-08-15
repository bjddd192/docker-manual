# argo-cd

Argo CD是Kubernetes的声明式GitOps持续交付工具。

应用程序定义、配置和环境应该是声明性的并且是版本控制的。应用程序部署和生命周期管理应该是自动化的、可审计的并且易于理解。

[官网](https://argoproj.github.io/argo-cd/)

[argoproj/argo-cd github](https://github.com/argoproj/argo-cd)

[argoproj/argocd-example-apps github](https://github.com/argoproj/argocd-example-apps)

[Best Practices](https://argoproj.github.io/argo-cd/user-guide/best_practices/)

### 安装 argo-cli

```sh
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x /usr/local/bin/argocd
argocd version
```

### 安装 argo-cli(mac)

```sh
VERSION=$(curl --silent "https://api.github.com/repos/argoproj/argo-cd/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*/\1/')
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/$VERSION/argocd-darwin-amd64
chmod +x /usr/local/bin/argocd
argocd version
```

### 添加多集群支持

在argocd中添加k8s集群无法在web控制台操作。默认情况下只能添加argocd所在的K8S集群。

如果要多集群管理支持，需要借助命令行处理。

[Argocd cluster add](https://argo-cd.readthedocs.io/en/stable/user-guide/commands/argocd_cluster_add/)

```sh
# 登录 argocd 控制台
argocd login scm-argocd.lesoon.com
# WARNING: server certificate had error: x509: certificate is valid for ingress.local, not scm-argocd.lesoon.com. Proceed insecurely (y/n)? y
# WARN[0001] Failed to invoke grpc call. Use flag --grpc-web in grpc calls. To avoid this warning message, use flag --grpc-web. 
# Username: admin
# Password: A123456
# 'admin:login' logged in successfully
# Context 'scm-argocd.lesoon.com' updated
# 查看集群信息
argocd cluster list -o json
# 添加一个外部集群
argocd cluster add kind-kind --kubeconfig my_cluster/config_30_199 --name kind-kind
# 查看所有集群
argocd cluster list
```

[argocd_cluster_add](https://argo-cd.readthedocs.io/en/stable/user-guide/commands/argocd_cluster_add/)

### 参考资料

[Tekton 与 Argo CD 结合实现 GitOps](https://mp.weixin.qq.com/s?__biz=MzU4MjQ0MTU4Ng==&mid=2247494240&idx=1&sn=3af78ac91ee04675d5e47622881cfd1e&chksm=fdbae57dcacd6c6b60d2a711c71083ce4eab63246a6d6c772180a70235494d0a987926696a0e&scene=178&cur_album_id=1905327982240399363#rd)

[更优雅的持续发布ArgoCD](https://www.sklinux.com/posts/k8s/argocd%E6%8C%81%E7%BB%AD%E5%8F%91%E5%B8%83/)

[Kubernetes之网络策略(Network Policy)](https://www.cnblogs.com/wangkun122/articles/12746842.html)

[GitOps最强工具-1. Argo CD](https://blog.csdn.net/weixin_37546425/article/details/105137283)

[GitOps最强工具-2. Argo CD安装](https://blog.csdn.net/weixin_37546425/article/details/105137539)

[GitOps最强工具-3. Argo CD部署应用](https://blog.csdn.net/weixin_37546425/article/details/105137736)

[在K8S中使用Argo CD做持续部署](https://cloud.tencent.com/developer/article/1750692)

[使用 GitLab CI 与 Argo CD 进行 GitOps 实践](https://www.qikqiak.com/post/gitlab-ci-argo-cd-gitops/)

[Argo CD 使用指南](https://kubeoperator.io/docs/user_manual/argocd/)
