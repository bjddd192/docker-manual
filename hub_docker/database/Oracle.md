# Oracle

## Oracle 11g

### 官方镜像

[Hub官方(已失效)](https://hub.docker.com/r/wnameless/oracle-xe-11g/)

[我的镜像](https://hub.docker.com/r/bjddd192/oracle-xe-11g)

[docker-oracle-xe-11g](https://hub.docker.com/r/deepdiver/docker-oracle-xe-11g)

[jaspeen/oracle-11g](https://hub.docker.com/r/jaspeen/oracle-11g)

[jinghongbo/oracle-11g](https://hub.docker.com/r/jinghongbo/oracle-11g)

### 启动命令

```sh
docker run --name oracle -d -p 1521:1521 --restart=always \
  -e ORACLE_ALLOW_REMOTE=true \
  bjddd192/oracle-xe-11g:18.04

# 数据目录持久化
mkdir -p /data/docker_volumn/oralce
docker cp oracle:/u01 /data/docker_volumn/oralce/u01

docker stop oracle && docker rm oracle

docker run --name oracle -d -p 60022:22 -p 1521:1521 -p 8080:8080 --restart=always \
  -e ORACLE_ALLOW_REMOTE=true \
  -v /data/docker_volumn/oralce/u01:/u01 \
  bjddd192/oracle-xe-11g:18.04
```

数据库连接信息:

```sh
hostname: 10.243.6.217   
port: 1521 
sid: xe 
username: system    
password: oracle    
```

jdbc连接信息:

```sh
jdbc:oracle:thin:@(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=10.243.6.217)(PORT=1521)))(CONNECT_DATA=(SERVER = DEDICATED)(SERVICE_NAME=xe)))
```

#### 国仓版本

```sh
docker run -itd --name oracle11204 -p 1521:1521 -p 222:22 -p 1158:1158 --restart=always \
  -h oracle11204 --privileged=true \
  hub.wonhigh.cn/tools/oracle_11g_ee_11.2.0.4:yangguocang init
  
# 进入容器
docker exec -it oracle11204 bash

# 启动数据库和监听
su - oracle -c 'lsnrctl start'
su - oracle -c 'echo "startup" | sqlplus / as sysdba'
su - oracle -c 'emctl start dbconsole'

# 目前数据持久化还有问题
# mkdir -p /data/docker_volumn/oralce
# docker cp oracle11204:/u01/app/oracle/oradata /data/docker_volumn/oralce/oradata

# docker stop oracle11204 && docker rm oracle11204
# 
# docker run -d --name oracle11204 -p 1521:1521 -p 222:22 -p 1158:1158 --restart=always \
#   -h oracle11204 --privileged=true \
#   -v /data/docker_volumn/oralce/oradata:/u01/app/oracle/oradata \
#   registry.cn-shenzhen.aliyuncs.com/yangguocang/oracle_11g_ee_11.2.0.4

hostname: 10.243.6.217   
port: 1521 
sid: orcl 
username: system    
password: oracle   
```

### 开启ssh

```sh
docker exec -it oracle bash
apt-get install openssh-server
apt-get install openssh-client
apt-get install vim
echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
service ssh start
```

### 服务器操作

```sql
-- 建立数据表空间
create tablespace ts_data_01 datafile  --创建名为ts_dat_dxs2的表空间文件
  '/u01/app/oracle/oradata/XE/ts_data_01.dbf'  --指定文件路径
size 500m  --指定文件大小
autoextend on  --自动扩展
next 500m  --数据文件满了以后扩展的大小
maxsize 30g  --数据文件的最大大小
logging  --声明这个表空间上所有的用户对象的日志属性（缺省是logging）。
online --改变表空间的状态。online使表空间创建后立即有效
permanent --表空间的属性:permanent永久表空间;temporary临时表空间;undo还原表空间
blocksize 8K  --设定一个不标准的块的大小,在临时表空间不能设置这个参数。
extent management local --表空间本地管理
uniform size 1M --说明表空间的范围的固定大小，缺省是1m
default nocompress  --指示存储不压缩
segment space management auto  --设置自动段空间管理
;

-- 建立临时表空间
create temporary tablespace ts_tmp_01 tempfile  --创建名为ts_tmp_dxs2的临时表空间文件
  '/u01/app/oracle/oradata/XE/ts_tmp_01.dbf'  --指定文件路径
size 100M  --指定文件大小
autoextend on  --自动扩展
next 100m  --数据文件满了以后扩展的大小
maxsize 500m  --数据文件的最大大小
extent management local --表空间本地管理
;

-- 查看表空间
select b.file_id "文件ID",
　　b.tablespace_name "表空间",
　　b.bytes/(1024*1024)  "总字节数(MB)",
　　(b.bytes-sum(nvl(a.bytes,0)))/(1024*1024) "已使用(MB)",
　　sum(nvl(a.bytes,0))/(1024*1024) "剩余(MB)",
　　round(sum(nvl(a.bytes,0))/(b.bytes)*100,2) "剩余百分比(%)",
　　b.file_name "物理文件名"
from dba_free_space a,dba_data_files b
where a.file_id=b.file_id
group by b.tablespace_name,b.file_name,b.file_id,b.bytes
order by b.tablespace_name,b.file_id;

-- 创建用户
create user usr_ho_scheduler 
identified by usr_ho_scheduler --密码
default tablespace ts_data_01 --默认表空间
temporary tablespace ts_tmp_01 --默认临时表空间
;

-- 赋权
grant connect,resource to usr_ho_scheduler; 
grant create any sequence to usr_ho_scheduler; 
grant create any table to usr_ho_scheduler; 
grant delete any table to usr_ho_scheduler; 
grant insert any table to usr_ho_scheduler; 
grant select any table to usr_ho_scheduler; 
grant unlimited tablespace to usr_ho_scheduler; 
grant execute any procedure to usr_ho_scheduler; 
grant update any table to usr_ho_scheduler; 
grant create any view to usr_ho_scheduler;

-- 重置密码
alter user usr_ho_scheduler identified by usr_ho_scheduler;
alter user usr_ho_scheduler account unlock;

-- 导出导入(未处理)
-- 查看导出路径
select * from dba_directories where directory_name='DATA_PUMP_DIR';

``` 
