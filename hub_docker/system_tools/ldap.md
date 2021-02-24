# ldap

LDAP统一认证服务。LDAP的开源实现是OpenLDAP，它比商业产品一点也不差，而且源码开放。

[openldap官网](https://www.openldap.org/)

### 官方镜像

[osixia/openldap](https://hub.docker.com/r/osixia/openldap)

### github

[osixia/docker-openldap](https://github.com/osixia/docker-openldap)

[osixia/docker-openldap-backup](https://github.com/osixia/docker-openldap-backup)

### 使用自定义证书

```sh
mkdir -p /data/docker_volumn/openldap/certs

cd /data/docker_volumn/openldap/certs

# ca配置文件
cat > /data/docker_volumn/openldap/certs/ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "ldap": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "87600h"
      }
    }
  }
}
EOF

# 自签名ca的证书申请
cat > /data/docker_volumn/openldap/certs/ldap-ca-csr.json << EOF
{
  "CN": "scm-ldap.lesoon.com",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Guangdong",
      "L": "Shenzhen",
      "O": "Lesoon",
      "OU": "DevOps"
    }
  ],
  "ca": {
    "expiry": "131400h"
  }
}
EOF

# ldap证书申请资料
# 下面hosts字段里就是使用这张证书的主机
# 特别注意一定要加上宿主机的IP地址，反正是自己颁发的证书，怎么加都行！！！
# 加上本机回环地址，加上ldap容器名，我这里容器名待会设置成openldap
# 如果你要放到公网去的话，那一可以加上FQDN地址
cat > ldap-csr.json << EOF
{
    "CN": "scm-ldap.lesoon.com",
    "hosts": [
      "127.0.0.1",
	  "10.0.43.27",
      "scm-ldap.lesoon.com"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
			"C": "CN",
	        "ST": "Guangdong",
	        "L": "Shenzhen",
	        "O": "Lesoon",
	        "OU": "DevOps"
        }
    ]
}
EOF

# CA自签名
cfssl gencert -initca ldap-ca-csr.json | cfssljson -bare ca

# LDAP证书签名,ldap需要的文件为：ca证书,ldap证书,ldap私钥
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=ldap ldap-csr.json | cfssljson -bare ldap

# 查看生成的证书
cfssl-certinfo -cert ldap-key.pem
cfssl-certinfo -cert ldap.pem
cfssl-certinfo -cert ca.pem
# 其中 ldap-key.pem ldap.pem ca.pem 是我们需要的
```

### 启动命令

```sh
# docker stop openldap && docker rm -f openldap
docker run --name openldap -d -p 389:389 -p 636:636 --restart=always \
--hostname scm-ldap.lesoon.com \
-e LDAP_ORGANISATION=lesoon \
-e LDAP_DOMAIN=lesoon.com \
-e LDAP_ADMIN_PASSWORD=Sa2VtYrEY9 \
-e LDAP_TLS=true \
-e LDAP_TLS_CRT_FILENAME=ldap.pem \
-e LDAP_TLS_KEY_FILENAME=ldap-key.pem \
-e LDAP_TLS_DH_PARAM_FILENAME=dhparam.pem \
-e LDAP_TLS_CA_CRT_FILENAME=ca.pem \
-e LDAP_TLS_ENFORCE=false \
-e LDAP_TLS_CIPHER_SUITE=SECURE256:-VERS-SSL3.0 \
-e LDAP_TLS_VERIFY_CLIENT=try \
-e TZ=Asia/Shanghai \
-e LDAP_BACKUP_CONFIG_CRON_EXP="0 5 * * *" \
-e LDAP_BACKUP_DATA_CRON_EXP="0 5 * * *" \
-e LDAP_BACKUP_TTL=15 \
-v /data/docker_volumn/openldap/database:/var/lib/ldap \
-v /data/docker_volumn/openldap/config:/etc/ldap/slapd.d \
-v /data/docker_volumn/openldap/certs:/container/service/slapd/assets/certs \
-v /data/docker_volumn/openldap/backup:/data/backup \
osixia/openldap-backup:1.4.0

# 管理工具部署
# docker stop phpldapadmin && docker rm -f phpldapadmin
docker run --name phpldapadmin -d -p 10080:80 -p 10443:443 --restart=always \
-e PHPLDAPADMIN_HTTPS=true \
-e PHPLDAPADMIN_HTTPS_CRT_FILENAME=ldap.pem \
-e PHPLDAPADMIN_HTTPS_KEY_FILENAME=ldap-key.pem \
-e PHPLDAPADMIN_HTTPS_CA_CRT_FILENAME=ca.pem \
-e PHPLDAPADMIN_LDAP_HOSTS=scm-ldap.lesoon.com \
-v /data/docker_volumn/openldap/certs:/container/service/phpldapadmin/assets/apache2/certs \
osixia/phpldapadmin:0.9.0

# docker stop lam && docker rm -f lam
docker run --name lam -d -p 20080:80 --restart=always \
-e PHPLDAPADMIN_HTTPS=false \
-e PHPLDAPADMIN_LDAP_HOSTS=scm-ldap.lesoon.com \
ldapaccountmanager/lam:7.4

# 验证，在浏览器访问，如：
https://10.0.43.27:10443/
login DN ： cn=admin,dc=lesoon,dc=com
Password ： Sa2VtYrEY9

# 客户端连接：
ldap://scm-ldap.lesoon.com:389
ldaps://scm-ldap.lesoon.com:636

docker exec -it openldap bash
# 验证配置文件
slaptest -u
# 查询用户
ldapsearch -LL -H ldapi:/// -D "cn=admin,dc=lesoon,dc=com" -W "uid=170992101" -b dc=lesoon,dc=com memberOf
```

### 使用Self Service Password管理用户密码

Self Service Password是一个PHP的Web应用，能够让用户修改在LDAP中的密码，支持标准的LDAPv3目录OpenLDAP, OpenDS, ApacheDS, Active Directory等。

[tiredofit/self-service-password(dockerhub)](https://hub.docker.com/r/tiredofit/self-service-password)

[tiredofit/docker-self-service-password(github)](https://github.com/tiredofit/docker-self-service-password)

[ltb-project/self-service-password(github)](https://github.com/ltb-project/self-service-password)

```sh
docker stop self-service-password && docker rm -f self-service-password

docker run --name self-service-password -d -p 8089:80 --restart=always \
-e LDAP_SERVER=ldap://scm-ldap.lesoon.com:389 \
-e LDAP_BINDDN=cn=admin,dc=lesoon,dc=com \
-e LDAP_BINDPASS=Sa2VtYrEY9 \
-e LDAP_BASE_SEARCH=ou=development,dc=lesoon,dc=com \
-e MAIL_FROM=wlmonitor@belle.com.cn \
-e SMTP_DEBUG=0 \
-e SMTP_HOST=smtp.exmail.qq.com \
-e SMTP_USER=wlmonitor@belle.com.cn \
-e SMTP_PASS=Wlmonitor@2017 \
-e SMTP_PORT=465 \
-e SMTP_SECURE_TYPE=ssl \
-e SMTP_AUTH_ON=true \
-e PASSWORD_MIN_LENGTH=8 \
-v /data/docker_volumn/self-service-password/data:/www/ssp \
-v /data/docker_volumn/self-service-password/logs:/www/logs \
tiredofit/self-service-password:5.1.2
```

#### 修复邮件的链接

```php
# php 脚本发送到邮件的重置密码链接在原本的代码里捕捉不到真正的请求域名
vim /data/docker_volumn/self-service-password/data/pages/sendtoken.php
#==============================================================================
# Send token by mail
#==============================================================================
if ( $result === "" ) {

    if ( empty($reset_url) ) {
        # 这里省略了
        
        # 调整的脚本
        $reset_url = $method."://".$_SERVER['HTTP_HOST'].$script_name;
        #$reset_url = $method."://".$server_name.$script_name;
    }

   # 这里省略了
}
```

### 禁止匿名访问

将以下信息导入openldap中：

```sh
docker exec -it openldap bash

cat > /root/disable_anon.ldif << "EOF"
dn: cn=config
changetype: modify
add: olcDisallows
olcDisallows: bind_anon

dn: cn=config
changetype: modify
add: olcRequires
olcRequires: authc

dn: olcDatabase={-1}frontend,cn=config
changetype: modify
add: olcRequires
olcRequires: authc
EOF

ldapadd -Y EXTERNAL -H ldapi:/// -f /root/disable_anon.ldif
```

### 多主复制

未实验成功，老是报证书错误，如：60221246 slap_client_connect: URI=ldap://scm-ldap2.lesoon.com Error, ldap_start_tls failed (-11)，暂时忽略。

#### TLS

```sh
mkdir -p /data/docker_volumn/openldap/certs
cd /data/docker_volumn/openldap/certs
# 创建根密钥
openssl genrsa -out ca.key 2048
# 创建自签名根证书
openssl req -x509 -new -nodes -key ca.key -sha256 -days 3650 \
-subj "/C=CN/ST=Guangdong/L=Shenzhen/O=Lesoon/OU=DevOps/CN=Ldap" \
-out ca.pem
# 查看自签名根证书
cfssl-certinfo -cert ca.pem
# 创建LDAP服务器私钥
openssl genrsa -out ldap.key 2048
# 创建证书签名请求
openssl req -new -key ldap.key -out ldap.csr
# You are about to be asked to enter information that will be incorporated
# into your certificate request.
# What you are about to enter is what is called a Distinguished Name or a DN.
# There are quite a few fields but you can leave some blank
# For some fields there will be a default value,
# If you enter '.', the field will be left blank.
# -----
# Country Name (2 letter code) [XX]:CN
# State or Province Name (full name) []:Guangdong
# Locality Name (eg, city) [Default City]:Shenzhen
# Organization Name (eg, company) [Default Company Ltd]:Lesoon
# Organizational Unit Name (eg, section) []:DevOps
# Common Name (eg, your name or your server's hostname) []:scm-ldap1.lesoon.com
# Email Address []:bjddd110@163.com
# 
# Please enter the following 'extra' attributes
# to be sent with your certificate request
# A challenge password []:
# An optional company name []:

# 使用自定义根CA签署证书签名请求
openssl x509 -req -in ldap.csr -CA ca.pem -CAkey ca.key -CAcreateserial -out ldap.crt -days 3650 -sha256
# 查看自签名证书
cfssl-certinfo -cert ldap.crt

docker stop openldap && docker rm -f openldap

docker run --name openldap -d -p 389:389 -p 636:636 --restart=always \
--add-host=scm-ldap1.lesoon.com:10.0.43.19 \
--add-host=scm-ldap2.lesoon.com:10.0.43.27 \
--hostname scm-ldap1.lesoon.com \
-e LDAP_ORGANISATION=lesoon \
-e LDAP_DOMAIN=lesoon.com \
-e LDAP_BASE_DN="dc=lesoon,dc=com" \
-e LDAP_ADMIN_PASSWORD=Sa2VtYrEY9 \
-e LDAP_TLS=true \
-e LDAP_TLS_CRT_FILENAME=ldap.crt \
-e LDAP_TLS_KEY_FILENAME=ldap.key \
-e LDAP_TLS_DH_PARAM_FILENAME=dhparam.pem \
-e LDAP_TLS_CA_CRT_FILENAME=ca.crt \
-e LDAP_TLS_ENFORCE=false \
-e LDAP_TLS_CIPHER_SUITE=SECURE256:-VERS-SSL3.0 \
-e LDAP_TLS_VERIFY_CLIENT=try \
-v /data/docker_volumn/openldap/database:/var/lib/ldap \
-v /data/docker_volumn/openldap/config:/etc/ldap/slapd.d \
-v /data/docker_volumn/openldap/certs:/container/service/slapd/assets/certs \
osixia/openldap:1.4.0


ldapsearch -H ldaps://scm-ldap.lesoon.com -d 1  -b o=lesoon.com -D "" -s base "(objectclass=*)"
```

10.0.43.19

```sh
docker stop openldap && docker rm -f openldap

docker run --name openldap -d --net=host --restart=always \
--hostname scm-ldap2.lesoon.com \
--add-host=scm-ldap1.lesoon.com:10.0.43.19 \
-e LDAP_ORGANISATION=lesoon \
-e LDAP_DOMAIN=lesoon.com \
-e LDAP_ADMIN_PASSWORD=Sa2VtYrEY9 \
-e LDAP_TLS_VERIFY_CLIENT=try \
-e LDAP_REPLICATION=true \
-e LDAP_REPLICATION_HOSTS="#PYTHON2BASH:['ldap://scm-ldap1.lesoon.com','ldap://scm-ldap2.lesoon.com']" \
-v /data/docker_volumn/openldap/database:/var/lib/ldap \
-v /data/docker_volumn/openldap/config:/etc/ldap/slapd.d \
osixia/openldap:1.4.0

docker stop openldap && docker rm -f openldap

docker run --name openldap -d --net=host --restart=always \
--hostname scm-ldap1.lesoon.com \
--add-host=scm-ldap2.lesoon.com:10.0.43.27 \
-e LDAP_ORGANISATION=lesoon \
-e LDAP_DOMAIN=lesoon.com \
-e LDAP_ADMIN_PASSWORD=Sa2VtYrEY9 \
-e LDAP_TLS_VERIFY_CLIENT=try \
-e LDAP_REPLICATION=true \
-e LDAP_REPLICATION_HOSTS="#PYTHON2BASH:['ldap://scm-ldap1.lesoon.com','ldap://scm-ldap2.lesoon.com']" \
-v /data/docker_volumn/openldap/database:/var/lib/ldap \
-v /data/docker_volumn/openldap/config:/etc/ldap/slapd.d \
osixia/openldap:1.4.0

docker stop openldap && docker rm -f openldap

docker stop phpldapadmin && docker rm -f phpldapadmin

docker run --name phpldapadmin -d -p 10080:80 --restart=always \
-e PHPLDAPADMIN_HTTPS=false \
-e PHPLDAPADMIN_LDAP_HOSTS=scm-ldap1.lesoon.com \
osixia/phpldapadmin:0.9.0

cd /data/docker_volumn/openldap/certs

# ca配置文件
cat > /data/docker_volumn/openldap/certs/ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "ldap": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "87600h"
      }
    }
  }
}
EOF

# 自签名ca的证书申请
cat > /data/docker_volumn/openldap/certs/ldap-ca-csr.json << EOF
{
  "CN": "*.lesoon.com",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Shenzhen",
      "L": "Shenzhen",
      "O": "ldap",
      "OU": "LDAP Security"
    }
  ],
  "ca": {
    "expiry": "131400h"
  }
}
EOF

# ldap证书申请资料
# 下面hosts字段里就是使用这张证书的主机
# 特别注意一定要加上宿主机的IP地址，反正是自己颁发的证书，怎么加都行！！！
# 加上本机回环地址，加上ldap容器名，我这里容器名待会设置成openldap
# 如果你要放到公网去的话，那一可以加上FQDN地址
cat > ldap-csr.json << EOF
{
    "CN": "*.lesoon.com",
    "hosts": [
      "127.0.0.1",
      "10.0.43.19",
	  "10.0.43.27",
      "scm-ldap1.lesoon.com",
      "scm-ldap2.lesoon.com"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "Shenzhen",
            "L": "Shenzhen",
            "O": "ldap",
            "OU": "LDAP Security"
        }
    ]
}
EOF

# CA自签名
cfssl gencert -initca ldap-ca-csr.json | cfssljson -bare ca

# LDAP证书签名,ldap需要的文件为：ca证书,ldap证书,ldap私钥
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=ldap ldap-csr.json | cfssljson -bare ldap

scp -P 60777 -r 10.0.43.27:/data/docker_volumn/openldap/certs/ /data/docker_volumn/openldap/

docker stop openldap && docker rm -f openldap

docker run --name openldap -d --net=host --restart=always \
--hostname scm-ldap1.lesoon.com \
-e LDAP_ORGANISATION=lesoon \
-e LDAP_DOMAIN=lesoon.com \
-e LDAP_ADMIN_PASSWORD=Sa2VtYrEY9 \
-e LDAP_TLS=true \
-e LDAP_TLS_CRT_FILENAME=ldap.pem \
-e LDAP_TLS_KEY_FILENAME=ldap-key.pem \
-e LDAP_TLS_DH_PARAM_FILENAME=dhparam.pem \
-e LDAP_TLS_CA_CRT_FILENAME=ca.pem \
-e LDAP_TLS_ENFORCE=false \
-e LDAP_TLS_CIPHER_SUITE=SECURE256:-VERS-SSL3.0 \
-e LDAP_TLS_VERIFY_CLIENT=try \
-e LDAP_REPLICATION=true \
-e LDAP_REPLICATION_HOSTS="#PYTHON2BASH:['ldap://scm-ldap1.lesoon.com','ldap://scm-ldap2.lesoon.com']" \
-v /data/docker_volumn/openldap/database:/var/lib/ldap \
-v /data/docker_volumn/openldap/config:/etc/ldap/slapd.d \
-v /data/docker_volumn/openldap/certs:/container/service/slapd/assets/certs \
osixia/openldap:1.4.0
```

10.0.43.27

```sh
docker run --name openldap -d --net=host --restart=always \
--hostname scm-ldap2.lesoon.com \
-e LDAP_ORGANISATION=lesoon \
-e LDAP_DOMAIN=lesoon.com \
-e LDAP_ADMIN_PASSWORD=Sa2VtYrEY9 \
-v /data/docker_volumn/openldap/database:/var/lib/ldap \
-v /data/docker_volumn/openldap/config:/etc/ldap/slapd.d \
-v /data/docker_volumn/openldap/certs:/container/service/slapd/assets/certs \
osixia/openldap:1.4.0

docker stop openldap && docker rm -f openldap

cd /data/docker_volumn/openldap/certs

# ca配置文件
cat > /data/docker_volumn/openldap/certs/ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "ldap": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "87600h"
      }
    }
  }
}
EOF

# 自签名ca的证书申请
cat > /data/docker_volumn/openldap/certs/ldap-ca-csr.json << EOF
{
  "CN": "*.lesoon.com",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "Shenzhen",
      "L": "Shenzhen",
      "O": "ldap",
      "OU": "LDAP Security"
    }
  ],
  "ca": {
    "expiry": "131400h"
  }
}
EOF

# ldap证书申请资料
# 下面hosts字段里就是使用这张证书的主机
# 特别注意一定要加上宿主机的IP地址，反正是自己颁发的证书，怎么加都行！！！
# 加上本机回环地址，加上ldap容器名，我这里容器名待会设置成openldap
# 如果你要放到公网去的话，那一可以加上FQDN地址
cat > ldap-csr.json << EOF
{
    "CN": "*.lesoon.com",
    "hosts": [
      "127.0.0.1",
      "10.0.43.19",
	  "10.0.43.27",
      "scm-ldap1.lesoon.com",
      "scm-ldap2.lesoon.com"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "Shenzhen",
            "L": "Shenzhen",
            "O": "ldap",
            "OU": "LDAP Security"
        }
    ]
}
EOF

# CA自签名
cfssl gencert -initca ldap-ca-csr.json | cfssljson -bare ca

# LDAP证书签名,ldap需要的文件为：ca证书,ldap证书,ldap私钥
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=ldap ldap-csr.json | cfssljson -bare ldap

docker stop openldap && docker rm -f openldap

docker run --name openldap -d -p 389:389 -p 636:636 --restart=always \
--add-host=scm-ldap1.lesoon.com:10.0.43.19 \
--add-host=scm-ldap2.lesoon.com:10.0.43.27 \
--hostname scm-ldap1.lesoon.com \
-e LDAP_ORGANISATION=lesoon \
-e LDAP_DOMAIN=lesoon.com \
-e LDAP_BASE_DN="dc=lesoon,dc=com" \
-e LDAP_ADMIN_PASSWORD=Sa2VtYrEY9 \
-e LDAP_TLS=true \
-e LDAP_TLS_CRT_FILENAME=ldap.crt \
-e LDAP_TLS_KEY_FILENAME=ldap.key \
-e LDAP_TLS_DH_PARAM_FILENAME=dhparam.pem \
-e LDAP_TLS_CA_CRT_FILENAME=ca.crt \
-e LDAP_TLS_ENFORCE=false \
-e LDAP_TLS_CIPHER_SUITE=SECURE256:-VERS-SSL3.0 \
-e LDAP_TLS_VERIFY_CLIENT=try \
-v /data/docker_volumn/openldap/database:/var/lib/ldap \
-v /data/docker_volumn/openldap/config:/etc/ldap/slapd.d \
-v /data/docker_volumn/openldap/certs:/container/service/slapd/assets/certs \
osixia/openldap:1.4.0


ldapsearch -ZZ -H ldap://scm-ldap1.lesoon.com -d 1  -b o=lesoon.com -D "" -s base "(objectclass=*)"

-e LDAP_REPLICATION=true \
-e LDAP_REPLICATION_HOSTS="#PYTHON2BASH:['ldap://scm-ldap1.lesoon.com','ldap://scm-ldap2.lesoon.com']" \
```

### 日志级别

输入命令slapd -d ?可以看到OpenLDAP预定义的日志级别以及每种日志级别所对应的数字（十进制和十六进制），如：

```sh
[ldap@192.168.121.130 ~$]slapd -d ?
Installed log subsystems:
Any (-1, 0xffffffff)
Trace (1, 0x1)
Packets (2, 0x2)
Args (4, 0x4)
Conns (8, 0x8)
BER (16, 0x10)
Filter (32, 0x20)
Config (64, 0x40)
ACL (128, 0x80)
Stats (256, 0x100)
Stats2 (512, 0x200)
Shell (1024, 0x400)
Parse (2048, 0x800)
Sync (16384, 0x4000)
None (32768, 0x8000)
NOTE: custom log subsystems may be later installed by specific code
```

### 参考资料

[openldap介绍和使用](https://cloud.tencent.com/developer/article/1490857)

[openLDAP入门与安装](https://blog.csdn.net/dikentoujing99/article/details/86501752)

[LDAP使用docker安装部署与使用](https://blog.csdn.net/weixin_42257195/article/details/102769495)

[烂泥：OpenLDAP安装与配置](https://www.ilanni.com/?p=13775)

[我花了一个五一终于搞懂了OpenLDAP](https://segmentfault.com/a/1190000014683418)

[jumpserver配置LDAP](https://blog.csdn.net/weixin_50462604/article/details/108727441)

[JumpServer中，LDAP服务器端配置和新增组和用户](https://blog.csdn.net/qq_40673345/article/details/104985016)

[jenkins集成ldap](https://www.cnblogs.com/zhaojiedi1992/p/zhaojiedi_liunx_52_ldap_for_jenkins.html)

[Centos7 搭建LDAP并启用TLS加密](https://blog.51cto.com/bigboss/2341986)

[cfssl自签CA证书用于TLS/SSL认证技术指南](https://segmentfault.com/a/1190000038276488)

[OpenLDAP 启用tls并授权rancher登录](http://kingsd.top/2020/03/04/openldap-enable-tls-and-authorize-rancher-login/)

[烂泥：openldap数据备份与恢复](https://www.ilanni.com/?p=14065)

[How to restore backups](https://github.com/osixia/docker-openldap-backup/issues/5)
