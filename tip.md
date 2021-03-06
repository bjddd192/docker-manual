# 小技巧

## 在一个已经 "exit” 的docker容器中修改配置文件

**此法验证失败，待进一步寻求方法。**

有时候可能需要修改运行在docker容器中的nginx的配置文件，或者其他一些已经运行和启动很久的容器中的配置文件。但是在这个过程可能稍有不慎，配置文件中，比如少了个分号，直接导致容器运行不起来，每次启动到一半就报错退出，使容器不可用。有没有一种办法可以在一个已经 "exit”的docker容器中修改配置文件呢？答案还是有的。

1. 运行命令 docker inspect [CONTAINER ID] 
2. 复制 MergedDir 中的路径，切换到对应的目录下

可以惊奇地发现前者的目录文件和直接进入容器中的可以看到的目录文件是一样的，这样我们就可以在前者中，也就是在容器已经 "exit" 的情况下，修改容器中的配置文件，比如 nginx 的话，就进入到 etc/nginx/ 下边去修改那个“忘记添加分号的” nginx.conf。

## 修改docker -v 挂载的文件遇到的问题

在启动docker容器时，为了保证一些基础配置与宿主机保持同步，通常需要将这些配置文件挂载进 docker容器，例如/etc/resolv.conf//etc/hosts//etc/localtime等。

当这些配置变化时，我们通常会修改这些文件。但是此时遇到了一个问题：

当在宿主机上修改这些文件后，docker容器内查看时，这些文件并未发生对应的修改。

然后通过查阅相关资料，发现该问题是由docker -v挂载文件和某些编辑器存储文件的行为共同导致 的。

- docker 挂载文件时，并不是挂载了某个文件的路径，而是实打实的挂载了对应的文件，即挂载了某 个指定的inode文件。
- 某些编辑器(vi)在编辑保存文件时，采用了备份、替换的策略，即编辑过程中，将变更写入新文件， 保存时，再将备份文件替换原文件，此时会导致文件的inode发生变化。
- 原inode对应的文件其实并没有发生修改。

因此，我们从宿主机上修改这些文件时，应该采用echo重定向等操作，避免文件的inode发生变化。

## docker容器中如何获取宿主机IP，连接宿主机的某个服务

1.通过环境变量传入docker run --env HOST_IP=192.168.0.160，通过环境变量 $HOST_IP 获取

例如：docker run --rm -it -e HOST_IP=`hostname -i` bjddd192/nginx:1.10.1 bash

2.运行docker时绑定hostdocker run --network host，通过ip route获取

## Docker之 /etc/profile 不生效得问题

我们启动服务后，发现配置在/etc/proifle得配置不生效，后来发现，容器里面执行命令，也没有生效，日了狗了，后来得解决方案，是将环境变量配置到 /root/.bashrc 这个里面 ，下次容器启动，环境变量也生效。

## 参考资料

[理解 inode](http://www.ruanyifeng.com/blog/2011/12/inode.html)

[docker重新进入容器时“/etc/profile”中环境变量失效问题的解决](https://blog.csdn.net/dongdong9223/article/details/81094657)
