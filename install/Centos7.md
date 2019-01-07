# Centos7 安装 docker

在一台干净的机器上安装 docker，使用阿里云的仓库，加快安装的过程。

```sh
# step 1: 安装必要的一些系统工具
yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装 Docker-CE
yum makecache fast
# 安装指定版本的Docker-CE:
yum list docker-ce.x86_64 --showduplicates | sort -r
yum -y install docker-ce-18.03.0.ce-1.el7.centos
docker --version
# Step 4: 重定义对docker0网桥，设置镜像加速器和信任仓库
tee /etc/docker/daemon.json <<-'EOF'
{
  "ip-forward": true,
  "bip": "10.23.10.1/24",
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "https://ekj7ys2t.mirror.aliyuncs.com",
    "https://docker.mirrors.ustc.edu.cn"
  ],
  "insecure-registries": [
    "hub.wonhigh.cn",
    "yougou.docker",
    "registry.eyd.com:5000"
  ],
  "max-concurrent-downloads": 10,
  "log-driver": "json-file",
  "log-level": "info",
  "log-opts": {
    "max-size": "50m",
    "max-file": "3"
  }
}
EOF
# Step 4: 开启Docker服务
systemctl start docker
# Step 5: 设置Docker服务开机自启动
systemctl enable docker
# Step 6: 指定harbor仓库地址
echo "10.234.6.219 hub.wonhigh.cn" >> /etc/hosts
# Step 7: 测试镜像拉取
docker pull hub.wonhigh.cn/retail/retail-gms-web:1.7.0-SNAPSHOT
# 注意：
# 官方软件源默认启用了最新的软件，您可以通过编辑软件源的方式获取各个版本的软件包。例如官方并没有将测试版本的软件源置为可用，您可以通过以下方式开启。同理可以开启各种测试版本等。
# vim /etc/yum.repos.d/docker-ee.repo
#   将[docker-ce-test]下方的enabled=0修改为enabled=1
#
# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# yum list docker-ce.x86_64 --showduplicates | sort -r
#   Loading mirror speeds from cached hostfile
#   Loaded plugins: branch, fastestmirror, langpacks
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            @docker-ce-stable
#   docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
#   Available Packages
# Step2: 安装指定版本的Docker-CE: (VERSION例如上面的17.03.0.ce.1-1.el7.centos)
# sudo yum -y install docker-ce-[VERSION]
```