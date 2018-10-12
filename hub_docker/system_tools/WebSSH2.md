# WebSSH2

[psharkey/webssh2](https://hub.docker.com/r/psharkey/webssh2/)



docker run --rm -it --name webssh2 -p 2222:2222 psharkey/webssh2

https://172.20.32.47/?ssh=ssh://root@172.20.32.47

http://172.20.32.47:2222/ssh/host/172.20.32.47

http://172.20.32.36:2222/ssh/host/172.20.32.36

```sh
# 添加一个用户
adduser k8sloger
passwd k8sloger 

# 使用 rbash 限制用户部分权限
ln -s /bin/bash /bin/rbash
bash -c 'echo "/bin/rbash" >> /etc/shells'
chsh -s /bin/rbash k8sloger

# 回收所有的权限
bash -c 'echo "export PATH=/home/k8sloger/" >> /home/k8sloger/.bashrc'

ln -s /bin/ping /home/k8sloger/ping
ln -s /bin/ls /home/k8sloger/ls
ln -s /bin/grep /home/k8sloger/grep
ln -s /bin/wc /home/k8sloger/wc
ln -s /bin/awk /home/k8sloger/awk
ln -s /bin/sudo /home/k8sloger/sudo

tee /home/k8sloger/getlog <<-'EOF'
#!/bin/sh

#sudo /root/local/bin/kubectl get namespace

read -p "请输入工程名:  " val 
result=$(sudo /root/local/bin/kubectl get pods --all-namespaces -o=wide | grep Running | grep $val | awk '{printf("k8s%03d %s\n", NR, $0)}')

# 如果是echo $result，输出结果为一行，没有换行符
# 如果是echo "$result"，输出结果为多行，有换行符
# echo "$result"

count=$(echo "$result" | wc -l)
#echo $count
#echo "$result"

if [ "$result" = "" ]
then
        echo "未发现 Running 的POD，请检查"
else
        if [ "$count" = "1" ]
        then
                echo "发现一个 Running 的POD，即将输出日志..."
                namespace=$(echo "$result" | awk '{print $2}')
                podname=$(echo "$result" | awk '{print $3}')
                sudo /root/local/bin/kubectl logs -f --tail 100 $podname  -n $namespace
        else
                # 如果是echo $result，输出结果为一行，没有换行符
                # 如果是echo "$result"，输出结果为多行，有换行符
                echo "$result"
                read -p "发现多个 Running 的POD，请输入 POD 序号:  " val
                podname=$(echo "$result" | grep $val | awk '{print $3}')
                namespace=$(echo "$result" | grep $val | awk '{print $2}')
                if [ -n "$namespace" ]
                then
                        sudo /root/local/bin/kubectl logs -f --tail 100 $podname  -n $namespace
                else
                        echo "未发现 Running 的POD，请检查"
                fi
        fi
fi
EOF

chmod +x /home/k8sloger/getlog
```

k8sloger   ALL=NOPASSWD:  /root/local/bin/kubectl

## 参考资料

[一个可以在浏览器上运行的SSH客户端：WebSSH2安装教程](https://www.moerats.com/archives/467/)

[想建一个用户只能执行几条命令](http://bbs.51cto.com/thread-1094531-1.html)

[rbash限制用户执行的命令](https://zzhaolei.github.io/2018/05/06/rbash%E9%99%90%E5%88%B6%E7%94%A8%E6%88%B7%E6%89%A7%E8%A1%8C%E7%9A%84%E5%91%BD%E4%BB%A4/)

[rbash - 一个受限的Bash Shell用实际示例说明](https://www.howtoing.com/rbash-a-restricted-bash-shell-explained-with-practical-examples/)