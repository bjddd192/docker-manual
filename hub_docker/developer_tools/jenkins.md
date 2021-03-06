# jenkins

[jenkins 官网](https://jenkins.io/zh/)

[jenkins 中文社区](https://jenkins-zh.cn/)

[jenkins 历史版本下载](http://mirrors.jenkins.io/)

[jenkins LTS 变更日志](https://jenkins.io/zh/changelog-stable/)

[jenkins plugins download](http://updates.jenkins-ci.org/download/plugins/)

## 官方镜像

[Hub官方](https://hub.docker.com/r/jenkins/jenkins)

[Hub官方(旧版)](https://hub.docker.com/_/jenkins)

## 我的镜像

[我的镜像](https://cloud.docker.com/repository/docker/bjddd192/jenkins)

## maven settings 文件

/data/docker_volumn/jenkins/settings.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
    maven 配置文件(用于jenkins)
-->

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">

    <!-- 本地仓库。该值表示构建系统本地仓库的路径。其默认值为: ${user.home}/.m2/repository -->
    <localRepository>/var/jenkins_home/maven-repository</localRepository>

    <!-- Maven是否需要和用户交互以获得输入。-->
    <!-- 如果Maven需要和用户交互以获得输入，则设置成true，反之则应为false。-->  
    <!-- 默认为true。-->  
    <interactiveMode>true</interactiveMode>

    <!-- Maven是否需要使用plugin-registry.xml文件来管理插件版本。  -->  
    <!-- 如果设置为true，则在{user.home}/.m2下需要有一个plugin-registry.xml来对plugin的版本进行管理。-->  
    <!-- 默认为false。-->  
    <usePluginRegistry>false</usePluginRegistry>  

    <!-- 表示Maven是否需要在离线模式下运行。如果构建系统需要在离线模式下运行，则为true，默认为false。  -->  
    <!-- 当由于网络设置原因或者安全因素，构建服务器不能连接远程仓库的时候，该配置就十分有用。  -->  
    <offline>false</offline>  

    <!-- 当插件的组织Id（groupId）没有显式提供时，供搜寻插件组织Id（groupId）的列表。  -->  
    <!-- 该元素包含一个pluginGroup元素列表，每个子元素包含了一个组织Id（groupId）。  -->  
    <!-- 当我们使用某个插件，并且没有在命令行为其提供组织Id（groupId）的时候，Maven就会使用该列表。  -->  
    <!-- 默认情况下该列表包含了org.apache.maven.plugins。  -->  
    <pluginGroups/>

    <!-- 用来配置不同的代理，多代理profiles可以应对笔记本或移动设备的工作环境：通过简单的设置profile id就可以很容易的更换整个代理配置。  -->  
    <proxies/> 

    <!-- 配置服务端的一些设置。一些设置如安全证书不应该和pom.xml一起分发。这种类型的信息应该存在于构建服务器上的settings.xml文件中。 -->  
    <servers> 

        <!-- 服务器元素包含配置服务器时需要的信息  -->
        <server>

            <!-- 这是server的id（注意不是用户登陆的id），该id与distributionManagement中repository元素的id相匹配。 -->
            <!-- 说白了就是 nexus 的 Repository ID。 -->
            <id>release</id>

            <!-- 鉴权用户名。鉴权用户名和鉴权密码表示服务器认证所需要的登录名和密码。  -->
            <username>deployment</username>

            <!-- 鉴权密码 。鉴权用户名和鉴权密码表示服务器认证所需要的登录名和密码。  -->  
            <password><![CDATA[deployment@dev123]]></password>

            <!-- 鉴权时使用的私钥位置。和前两个元素类似，私钥位置和私钥密码指定了一个私钥的路径（默认是/home/hudson/.ssh/id_dsa）以及如果需要的话，一个密钥   -->
            <!-- 将来passphrase和password元素可能会被提取到外部，但目前它们必须在settings.xml文件以纯文本的形式声明。   --> 
            <!-- <privateKey>${usr.home}/.ssh/id_dsa</privateKey>   -->
                
            <!-- 鉴权时使用的私钥密码。 -->  
            <!-- <passphrase>some_passphrase</passphrase>  --> 

            <!-- 文件被创建时的权限。如果在部署的时候会创建一个仓库文件或者目录，这时候就可以使用权限（permission）。-->  
            <!-- 这两个元素合法的值是一个三位数字，其对应了unix文件系统的权限，如664，或者775。  -->  
            <!-- <filePermissions>664</filePermissions> -->  

            <!-- 目录被创建时的权限。  -->  
            <!-- <directoryPermissions>775</directoryPermissions>   -->
                
            <!-- 传输层额外的配置项  -->  
            <!-- <configuration></configuration>  --> 
        </server>

        <server>
            <id>snapshot</id>
            <username>deployment</username>
            <password><![CDATA[deployment@dev123]]></password>
        </server> 

        <server>
            <id>thirdparty</id>
            <username>deployment</username>
            <password><![CDATA[deployment@dev123]]></password>
        </server>

    </servers>

    <!-- 为仓库列表配置的下载镜像列表。-->  
    <mirrors>

        <!--给定仓库的下载镜像。  --> 
        <mirror>

            <!--该镜像的唯一标识符。id用来区分不同的mirror元素。  --> 
            <id>nexus</id>

            <!--镜像名称  -->  
            <name>local private nexus</name>

            <!--该镜像的URL。构建系统会优先考虑使用该URL，而非使用默认的服务器URL。 --> 
            <url>http://192.168.105.88:20020/nexus/content/groups/public</url>

            <!--镜像所包含的仓库的Id。例如，如果我们要设置了一个Maven中央仓库（http://repo1.maven.org/maven2）的镜像，-->  
            <!--就需要将该元素设置成central。这必须和中央仓库的id central完全一致。 --> 
            <mirrorOf>*</mirrorOf>

        </mirror>

    </mirrors>

    <!--根据环境参数来调整构建配置的列表。settings.xml中的profile元素是pom.xml中profile元素的裁剪版本。-->  
    <!--它包含了id，activation, repositories, pluginRepositories和 properties元素。-->  
    <!--这里的profile元素只包含这五个子元素是因为这里只关心构建系统这个整体（这正是settings.xml文件的角色定位），而非单独的项目对象模型设置。-->  
    <!--如果一个settings中的profile被激活，它的值会覆盖任何其它定义在POM中或者profile.xml中的带有相同id的profile。  -->
    <profiles>

        <!--根据环境参数来调整的构件的配置 -->
        <profile>

            <!--该配置的唯一标识符。  -->
            <id>nexus</id>

            <!--远程仓库列表，它是Maven用来填充构建系统本地仓库所使用的一组远程项目。  -->
            <repositories>

                <!--包含需要连接到远程仓库的信息  -->  
                <repository>

                    <!--远程仓库唯一标识 --> 
                    <id>nexus</id>

                    <!--远程仓库名称  -->  
                    <name>local private nexus</name>

                    <!--远程仓库的地址  -->
                    <url>http://192.168.105.88:20020/nexus/content/groups/public</url>

                </repository>

            </repositories>
        </profile>
    </profiles>

    <!--手动激活profiles的列表，按照profile被应用的顺序定义activeProfile。 该元素包含了一组activeProfile元素，每个activeProfile都含有一个profile id。-->  
    <!--任何在activeProfile中定义的profile id，不论环境设置如何，其对应的 profile都会被激活。-->  
    <!--如果没有匹配的profile，则什么都不会发生。例如，env-test是一个activeProfile，则在pom.xml（或者profile.xml）中对应id的profile会被激活。-->  
    <!--如果运行过程中找不到这样一个profile，Maven则会像往常一样运行。  -->  
    <activeProfiles>
        <activeProfile>nexus</activeProfile>
    </activeProfiles>

</settings>
```

## 启动命令

```sh
docker stop jenkins && docker rm jenkins

docker run --name jenkins -d -p 29999:8080 -p 5000:50000 --restart=always \
  --memory 2G -u root \
  -v /data/docker_volumn/jenkins:/var/jenkins_home \
  -v /usr/bin/docker:/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /usr/lib64/libltdl.so.7:/usr/lib/x86_64-linux-gnu/libltdl.so.7 \
  bjddd192/jenkins:2.165

# 拷贝出 /usr/share/jenkins 方便后续 jenkins 升级
docker cp jenkins:/usr/share/jenkins /data/docker_volumn/jenkins_war

# 生成 maven 配置文件
tee > /data/docker_volumn/jenkins/settings.xml <<'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <localRepository>/var/jenkins_home/maven-repository</localRepository>
    <interactiveMode>true</interactiveMode>
    <usePluginRegistry>false</usePluginRegistry>
    <offline>false</offline>
    <pluginGroups/>  
    <proxies/> 
    <servers> 
        <server>
            <id>release</id>
            <username>deployment</username>  
            <password><![CDATA[deployment@dev123]]></password> 
        </server>
        <server>
            <id>snapshot</id>
            <username>deployment</username>
            <password><![CDATA[deployment@dev123]]></password>
        </server> 
        <server>
            <id>thirdparty</id>
            <username>deployment</username>
            <password><![CDATA[deployment@dev123]]></password>
        </server>
            <server>
            <id>docker-hub</id>
            <username>scm</username>
            <password>n7izpoc6N2</password>
            <configuration>
                <email>scm@ex.com</email>
            </configuration>
        </server>
    </servers> 
    <mirrors>
        <mirror> 
            <id>nexus</id> 
            <name>local private nexus</name>
            <url>http://192.168.105.88:20020/nexus/content/groups/public</url> 
            <mirrorOf>*</mirrorOf>
        </mirror>
    </mirrors>
    <profiles>
        <profile>
            <id>nexus</id>
            <repositories>
                <repository>
                    <id>nexus</id>
                    <name>local private nexus</name>
                    <url>http://192.168.105.88:20020/nexus/content/groups/public</url>
                </repository>
            </repositories>
        </profile>
    </profiles> 
    <activeProfiles>
        <activeProfile>nexus</activeProfile>
    </activeProfiles>
</settings>
EOF
  
docker stop jenkins && docker rm jenkins

docker run --name jenkins -d -p 29999:8080 -p 5000:50000 --restart=always \
  --memory 3G -u root \
  -v /data/docker_volumn/jenkins:/var/jenkins_home \
  -v /data/docker_volumn/jenkins_war:/usr/share/jenkins \
  -v /usr/bin/docker:/usr/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v /usr/lib64/libltdl.so.7:/usr/lib/x86_64-linux-gnu/libltdl.so.7 \
  -v /data/docker_volumn/jenkins/settings.xml:/usr/local/apache-maven-3.6.0/conf/settings.xml \
  bjddd192/jenkins:2.165
```

## 验证

访问 url 如：http://serverIP:29999

## 2.199版本安装

```sh
docker pull dockerhub.azk8s.cn/jenkins/jenkins:2.199-centos
docker tag dockerhub.azk8s.cn/jenkins/jenkins:2.199-centos hub.wonhigh.cn/tools/jenkins:2.199-centos
docker push hub.wonhigh.cn/tools/jenkins:2.199-centos

docker stop jenkins && docker rm jenkins

docker run --name jenkins -d -p 29999:8080 -p 50000:50000 --restart=always \
  --memory 3G -u root \
  -v /usr/local/bin/docker:/usr/local/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  hub.wonhigh.cn/tools/jenkins:2.199-centos

# 拷贝出 /usr/share/jenkins 方便后续 jenkins 升级
mkdir -p /data/docker_volumn/jenkins
docker cp jenkins:/usr/share/jenkins /data/docker_volumn/jenkins/war
docker cp jenkins:/var/jenkins_home /data/docker_volumn/jenkins/data

# 修改 hudson.model.UpdateCenter.xml文件，或者界面设置插件更新地址：
# http://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json 

# 安装插件提速配置
sed -i 's/http:\/\/updates.jenkins-ci.org\/download/https:\/\/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' /data/docker_volumn/jenkins/data/updates/default.json 
sed -i 's/http:\/\/www.google.com/https:\/\/www.baidu.com/g' /data/docker_volumn/jenkins/data/updates/default.json

docker stop jenkins && docker rm jenkins

docker run --name jenkins -d -p 29999:8080 -p 50000:50000 --restart=always \
  --memory 3G -u root \
  -e TZ=Asia/Shanghai \
  -v /etc/localtime:/etc/localtime:ro \
  -v /data/docker_volumn/jenkins/data:/var/jenkins_home \
  -v /data/docker_volumn/jenkins/war:/usr/share/jenkins \
  -v /usr/local/bin/docker:/usr/local/bin/docker \
  -v /var/run/docker.sock:/var/run/docker.sock \
  hub.wonhigh.cn/tools/jenkins:2.199-centos

# 安装插件清单：
# 语言包 - Localization: Chinese (Simplified)
# git插件 - Git 、 Git client 、 Gitlab Hook 
# ssh插件 - SSH 、 SSH Credentials 
# email插件 - Email Extension 
# 钉钉插件 - DingTalk 
```

## 配置 Android Sdk 环境

[Android Studio](https://developer.android.com/studio/)

[platform-tools](https://developer.android.com/studio/releases/platform-tools)

```sh
wget http://dl.google.com/android/android-sdk_r24.4.1-linux.tgz
wget http://10.0.43.24:8066/package/android/android-sdk_r24.4.1-linux.tgz
tar xzf android-sdk_r24.4.1-linux.tgz

docker exec -it jenkins bash
android -v list sdk
echo y | android update sdk -u -a --filter 1,2,5,6,7,8
echo y | android update sdk -u -a --filter tools,platform-tools,android-26,android-27,android-28,android-29,build-tools-28.0.3,build-tools-29.0.2
```

[nodejs 下载](https://npm.taobao.org/mirrors/node/)

[gradle 下载](https://gradle.org/releases/)

## jenkins slave

[Hub官方](https://hub.docker.com/r/jenkins/jnlp-slave)

```sh
docker stop jenkins-slave && docker rm -f jenkins-slave

docker run --name jenkins-slave -d --restart=always jenkins/jnlp-slave:3.35-5-alpine -url http://172.17.209.53:29999 -workDir="/home/jenkins/agent" 56d05e920629bc1224f284bc4492b8997502ebd53ad0cb67fc705d34954325b3 test
```

## 课程资料

课程PPT地址：

链接:https://pan.baidu.com/s/1OSynEP3AUrjtq1BR2iZlLA  密码:azun  

课程文档:
https://github.com/zeyangli/jenkins_pipeline_docs.git  新版本文档【下载本地阅读更佳】[更新中]

http://zeyangli.github.io [视频中的旧版本文档]
 
[课程文档](http://119.3.228.122/jenkins/)

## 参考资料

[Jenkins 插件更新慢的问题](https://www.jianshu.com/p/5c6d0416bfc3)

[Jenkins安装插件提速](Jenkins安装插件提速)

[图文讲解jenkins的安装与配置---远程发布、自动监测代码更新](https://blog.csdn.net/zzq900503/article/details/41827481)

[Jenkins构建完成自动发送邮件](https://blog.csdn.net/lht3347/article/details/84325326)

[Docker + Jenkins + Cordova建设移动端打包平台](https://www.jianshu.com/p/c50423a3f40f)

[linux 安装 Android Sdk](https://www.jianshu.com/p/86b9c57bf838)

[Android报错之.android/repositories.cfg could not be loaded.解决方案](https://blog.csdn.net/u010358168/article/details/84827249)

[./sdkmanager --licenses Error: Unknown argument --licenses](https://blog.csdn.net/u010165147/article/details/82496617)

[jenkins 如何在 k8s 集群中实现动态 agent](https://blog.51cto.com/wzlinux/2467307?source=drh)

[动态 Jenkins Slave](https://www.qikqiak.com/k8s-book/docs/36.Jenkins%20Slave.html)

[Jenkins master and slave with docker](https://medium.com/@prashant.vats/jenkins-master-and-slave-with-docker-b993dd031cbd)

[在 Kubernetes 中通过 Jenkins 和 Dynamic Slaves 实现 CI/CD](https://laijingwu.com/440.html)

[jenkins中文乱码与服务启动](http://yeshaoting.github.io/article/jenkins/jenkins%E4%B8%AD%E6%96%87%E4%B9%B1%E7%A0%81%E4%B8%8E%E6%9C%8D%E5%8A%A1%E5%90%AF%E5%8A%A8/)
