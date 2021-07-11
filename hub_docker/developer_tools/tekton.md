# tekton

Tekton 是一个功能强大且灵活的Kubernetes 原生开源框架，用于创建持续集成和交付（CI/CD）系统。Tekton不能使你立即获得CICD的能力。但是基于Tekton可以设计出各种花式的构建部署流水线。得益于Tekton良好的抽象，这些设计出的流水线可以作为模板在多个组织，项目间共享。 

[官网](https://tekton.dev/)

### 使用步骤

[git-clone](https://hub.tekton.dev/tekton/task/git-clone)

```sh
cd /data/tekton/tekton-test/tasks

kubectl apply -f serviceaccount.yaml

# kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-clone/0.4/git-clone.yaml
# tkn hub install task git-clone
kubectl apply -f git-clone.yaml

yum -y install jq

kubectl create secret docker-registry dockerhub \
--docker-server=http://hub.wonhigh.cn --docker-username=scm --docker-password=n7izpoc6N2 \
--dry-run=client -o json | jq -r '.data.".dockerconfigjson"' | base64 -d > /tmp/config.json \
&& kubectl create secret generic docker-config --from-file=/tmp/config.json && rm -f /tmp/config.json

# kubectl delete -f run/run.yaml
# kubectl delete -f pipeline/build-pipeline.yaml
# kubectl delete -f tasks/deploy-to-k8s.yaml
# kubectl delete -f tasks/source-to-image.yaml

kubectl apply -f tasks/source-to-image.yaml

kubectl apply -f tasks/deploy-to-k8s.yaml

# 组装流水线
kubectl apply -f pipeline/build-pipeline.yaml

# 执行流水线
kubectl apply -f run/run.yaml
```

### kaniko

kaniko是一个在容器或Kubernetes集群中从Dockerfile构建容器映像的工具。

[GoogleContainerTools/kaniko](https://github.com/GoogleContainerTools/kaniko)

### 参考资料

[kubernetes原生CI/CD工具：Tekton探秘与上手实践](https://segmentfault.com/a/1190000020182215)

[云原生 CICD: Tekton Pipeline 实战](https://atbug.com/tekton-pipeline-practice/)

