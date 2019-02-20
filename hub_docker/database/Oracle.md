# Oracle

## Oracle 11g

### 官方镜像

[Hub官方(已失效)](https://hub.docker.com/r/wnameless/oracle-xe-11g/)

[我的镜像](https://hub.docker.com/r/bjddd192/oracle-xe-11g)

[docker-oracle-xe-11g](https://hub.docker.com/r/deepdiver/docker-oracle-xe-11g)

### 启动命令

```sh
docker run --name oracle -d -p 49161:1521 --restart=always \
  -e ORACLE_ALLOW_REMOTE=true \
  bjddd192/oracle-xe-11g:18.04

# 数据目录持久化未成功，待后面研究
# /data/docker_volumn/oralce/data:/u01/app/oracle/oradata/XE \
```

数据库连接信息:

```sh
hostname: localhost   
port: 49161 
sid: xe 
username: system    
password: oracle    
```

jdbc连接信息:

```sh
jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=10.0.43.32)(PORT=49161)))(CONNECT_DATA=(SERVER = DEDICATED)(SERVICE_NAME=xe)))
```
