# metersphere

MeterSphere 是一站式的开源企业级持续测试平台，涵盖测试跟踪、接口测试、性能测试、团队协作等功能，兼容JMeter 等开源标准，有效助力开发和测试团队充分利用云弹性进行高度可扩展的自动化测试，加速高质量软件的交付。

## 官方

[官网](https://www.fit2cloud.com/metersphere/index.html)

[官方文档](https://metersphere.io/docs/index.html)

[gitlab](https://github.com/metersphere/metersphere/)

## 安装

### 创建外部数据库

```sql
CREATE DATABASE db_metersphere CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
GRANT ALL PRIVILEGES ON `db_metersphere`.* TO 'user_ms' @'%' IDENTIFIED BY 'metersphere';
FLUSH PRIVILEGES;
```

### 启动命令

```sh
cd /data/metersphere
./quick_start.sh
cd /data/metersphere/metersphere-release-v1.2.0
# vi install.conf
# 改为外部数据库 
./install.sh

# 停止服务
cd /opt/metersphere
docker-compose $(cat compose_files) down -v
```

### 验证

```sh
地址: http://目标服务器IP地址:8081
用户名: admin
密码: metersphere
```
