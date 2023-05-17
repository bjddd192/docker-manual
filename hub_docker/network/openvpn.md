## openvpn

[kylemanna/docker-openvpn](https://github.com/kylemanna/docker-openvpn)

[kylemanna/openvpn](https://hub.docker.com/r/kylemanna/openvpn)

```sh
OVPN_DATA="ovpn-data"
docker volume create --name $OVPN_DATA
#配置
docker run -v $OVPN_DATA:/etc/openvpn --rm kylemanna/openvpn ovpn_genconfig -u udp://119.23.232.186
#密钥配置(123456)
docker run -v $OVPN_DATA:/etc/openvpn --rm -it kylemanna/openvpn ovpn_initpki
#启动
docker run -v $OVPN_DATA:/etc/openvpn -d --name openvpn -p 1194:1194/udp --cap-add=NET_ADMIN kylemanna/openvpn
#生成客户端.ovpn test为自定义名称
docker run -v $OVPN_DATA:/etc/openvpn --rm -it kylemanna/openvpn easyrsa build-client-full eclouda nopass
docker run -v $OVPN_DATA:/etc/openvpn --rm kylemanna/openvpn ovpn_getclient eclouda > /tmp/eclouda.ovpn
#使用scp把test.ovpn导到本地，使用openvpn客户端打开

#进入容器，修改配置
docker exec -it openvpn bash 
vi /etc/openvpn/openvpn.conf
push "route 192.168.0.0 255.255.255.0"
push "route 192.168.1.0 255.255.255.0"
push "route 192.168.2.0 255.255.255.0"
push "route 192.168.96.0 255.255.240.0"

#需要创建用户时运行以下命令
OVPN_DATA="ovpn-data"
#生成客户端.ovpn test为自定义名称
docker run -v /var/lib/docker/volumes/ovpn-data/_data:/etc/openvpn --rm -it kylemanna/openvpn easyrsa build-client-full yun.fy nopass
docker run -v /var/lib/docker/volumes/ovpn-data/_data:/etc/openvpn --rm kylemanna/openvpn ovpn_getclient yun.fy > yun.fy.ovpn
#使用scp把test.ovpn导到本地，使用openvpn客户端打开

docker run -v /var/lib/docker/volumes/ovpn-data/_data:/etc/openvpn --rm -it kylemanna/openvpn easyrsa revoke test 
docker run -v /var/lib/docker/volumes/ovpn-data/_data:/etc/openvpn --rm -it kylemanna/openvpn easyrsa gen-crl

```

[客户端下载(需翻墙)](https://openvpn.net/)

[阿里云使用Docker快速搭建内网openvpn](https://zoyi14.smartapps.cn/pages/note/index?slug=c0c8da51fbe6&origin=share&bdswankey=vivobrowser%3A%2F%2Fswan%2FfjESu3W8LB8fsE3tG3xUoMXSvvDjawbn%2Fpages%2Fnote%2Findex%3Fslug%3Dc0c8da51fbe6%26searchParams%3D%257B%2522failUrl%2522%253A%2522https%253A%255C%252F%255C%252Fwww.jianshu.com%255C%252Fp%255C%252Fc0c8da51fbe6%2522%257D%26useTpl%3D1&_swebfr=1&_swebFromHost=vivobrowser)

[企业内部openvpn快速入门搭建](https://zhuanlan.zhihu.com/p/440346670)

[在CentOS系统的ECS实例中如何配置OpenVPN](https://help.aliyun.com/document_detail/42521.html)

[暴露ACK网络](https://www.linux86.com/387.html)

[本机与云服务器内网互联](https://cloud.tencent.com/developer/article/2235303)

[Docker部署OpenVPN就是这么简单](http://www.legendwolf.com/?id=62)
