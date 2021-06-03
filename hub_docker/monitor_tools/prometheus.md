# prometheus 监控

### prometheus server

[官网](https://prometheus.io/)

[github](https://github.com/prometheus/prometheus)

[CHANGELOG](https://github.com/prometheus/prometheus/blob/master/CHANGELOG.md)

[hub官方](https://hub.docker.com/r/prom/prometheus)

```sh
docker run -d --name prometheus -p 9090:9090 --restart=always \
  prom/prometheus:v2.17.2

# 持久化数据
docker cp prometheus:/prometheus /data/docker_volumn/
docker cp prometheus:/etc/prometheus/prometheus.yml /data/docker_volumn/prometheus/prometheus.yml

# 调整 prometheus 配置
vi /data/docker_volumn/prometheus/prometheus.yml

docker stop prometheus && docker rm -f prometheus

# --storage.tsdb.retention 该参数决定何时删除旧数据，默认为15天。 
# --web.enable-admin-api    开放API,这样才能够，使用api删除数据
# --web.enable-lifecycle    开放重载配置，通过curl -X POST http://localhost:9090/-/reload
#- -web.external-url=http://xx.xxx.com  用于在推送告警时， 代表.GeneratorURL参数的值，方便直接访问Prometheus
docker run -d --name prometheus -p 9090:9090 --restart=always \
  -u root \
  -v /data/docker_volumn/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
  -v /data/docker_volumn/prometheus:/prometheus \
  prom/prometheus:v2.17.2 \
  --storage.tsdb.retention=60d \
  --web.enable-admin-api \
  --web.enable-lifecycle \
  --web.external-url="http://10.250.15.39:9090"

# 重载配置
curl -X POST http://10.250.15.39:9090/-/reload

# 删除所有数据(并不会立即删除数据，目前没感觉出什么用处)
# curl -X POST -g 'http://localhost:9090/api/v1/admin/tsdb/delete_series?match[]={__name__=~".+"}'

# 清理 DB
# curl -XPOST http://localhost:9090/api/v1/admin/tsdb/clean_tombstones
```

#### 数据清理

[Prometheus 删除数据指标](https://www.qikqiak.com/post/prometheus-delete-metrics/)

#### 参考资料

[Prometheus 入门与实践](https://www.ibm.com/developerworks/cn/cloud/library/cl-lo-prometheus-getting-started-and-practice/index.html)

[基于docker容器部署Prometheus服务——云平台监控利器](https://blog.51cto.com/14154700/2445258)

[Prometheus搭建部署及官网翻译](http://www.51niux.com/?id=244)

### 收集器 exporter

[prometheus数据采集exporter全家桶](https://blog.51cto.com/13053917/2374734)

#### node-exporter

[hub官方](https://hub.docker.com/r/prom/node-exporter)

```sh
docker stop node-exporter && docker rm -f node-exporter

# prom/node-exporter:v0.18.1
docker run -d --name node-exporter --net host --restart=always \
  -v /proc:/host/proc \
  -v /sys:/host/sys \
  -v /:/rootfs \
  harbor.bjds.belle.lan/k8s/node-exporter:v0.18.1 \
  --path.procfs /host/proc --path.sysfs /host/sys \
  --collector.filesystem.ignored-mount-points "^/(sys|proc|dev|host|etc)($|/)"

docker run -d --name node-exporter --net host --restart=always \
    -v /proc:/host/proc \
    -v /sys:/host/sys \
    -v /:/rootfs \
    prom/node-exporter:v0.18.1 \
    --path.procfs /host/proc --path.sysfs /host/sys \
    --collector.filesystem.ignored-mount-points "^/(sys|proc|dev|host|etc)($|/)"
```

##### 参考资料

[Prometheus Node_Exporter监控主机性能展示Grafana](http://www.idcsec.com/2018/12/29/Prometheus-Node-Exporter%E7%9B%91%E6%8E%A7%E4%B8%BB%E6%9C%BA%E6%80%A7%E8%83%BD%E5%B1%95%E7%A4%BAGrafana/)

#### blackbox-exporter

blackbox_exporter 是用来监控http, tcp, dns, icmp协议的工具。

[hub官方](https://hub.docker.com/r/prom/blackbox-exporter)

[github](https://github.com/prometheus/blackbox_exporter)

```sh
docker run -d --name blackbox-exporter -p 9115:9115 --restart=always \
  prom/blackbox-exporter:v0.16.0

# 持久化数据
docker cp blackbox-exporter:/etc/blackbox_exporter /data/docker_volumn/

# 调整配置
vi /data/docker_volumn/blackbox_exporter/config.yml

docker stop blackbox-exporter && docker rm -f blackbox-exporter

docker run -d --name blackbox-exporter -p 9115:9115 --restart=always \
  -v /data/docker_volumn/blackbox_exporter:/etc/blackbox_exporter \
  prom/blackbox-exporter:v0.16.0

# 验证 ping 接口
# http://172.17.209.202:9115/probe?module=ping&target=172.17.209.53
```

#### clickhouse-exporter

[hub官方](https://hub.docker.com/r/f1yegor/clickhouse-exporter)

[github](https://github.com/f1yegor/clickhouse_exporter)

```sh
docker run -d --name clickhouse-exporter -p 9116:9116 --restart=always \
-e CLICKHOUSE_USER=default \
-e CLICKHOUSE_PASSWORD=go2House \
f1yegor/clickhouse-exporter -scrape_uri=http://172.17.209.53:51011/
```



##### 参考资料

[prometheus的黑盒监控](https://zhangguanzhang.github.io/2018/12/04/black-box-exporter/)

[Kubernetes + Blackbox 实现对 Web 和 DNS 的简单监控](https://blog.fleeto.us/post/blackbox-monitor-dns-web/)

[网络探测：Blackbox Exporter](https://blog.51cto.com/13447608/2469397)

[使用Prometheus的blackbox_exporter进行网络监控](http://www.idcsec.com/2018/10/28/%E4%BD%BF%E7%94%A8Prometheus%E7%9A%84blackbox-exporter%E8%BF%9B%E8%A1%8C%E7%BD%91%E7%BB%9C%E7%9B%91%E6%8E%A7/)

[使用Prometheus的blackbox_exporter进行网络监控](https://blog.frognew.com/2018/02/prometheus-blackbox-exporter.html)

[用Prometheus进行网络质量ping监控Grafana进行监控数据展示](https://www.iamle.com/archives/2130.html)

[blackbox_exporter/example.yml](https://github.com/prometheus/blackbox_exporter/blob/master/example.yml)

[Prometheus Operator 常用指标](https://cloud.tencent.com/developer/article/1667912)

### 数据看板 grafana

[官网](https://grafana.com/)

[github](https://github.com/grafana/grafana)

[CHANGELOG](https://github.com/grafana/grafana/blob/master/CHANGELOG.md)

[hub官方](https://hub.docker.com/r/grafana/grafana)

```sh
docker run -d --name grafana -p 3000:3000 --restart=always \
   grafana/grafana:6.7.3

# 持久化数据
mkdir -p /data/docker_volumn/prometheus_grafana/share
mkdir -p /data/docker_volumn/prometheus_grafana/lib
docker cp grafana:/usr/share/grafana /data/docker_volumn/prometheus_grafana/share/
docker cp grafana:/var/lib/grafana /data/docker_volumn/prometheus_grafana/lib/

docker stop grafana && docker rm -f grafana

docker run -d --name grafana -p 3000:3000 --restart=always \
  --memory 4G -u root \
  -v /data/docker_volumn/prometheus_grafana/share/grafana:/usr/share/grafana \
  -v /data/docker_volumn/prometheus_grafana/lib/grafana:/var/lib/grafana \
  grafana/grafana:6.7.3

# 安装 grafana-piechart-panel 饼图插件(可能失败，多试两次)
docker exec -it grafana grafana-cli plugins install grafana-piechart-panel 1.3.8
docker restart grafana
```

- node-exporter 看板ID： https://grafana.com/grafana/dashboards/8919
- blackbox-exporter 看板ID： 
- clickhouse-exporter 看板ID： https://grafana.com/grafana/dashboards/882

#### 优质看板

[NGINX Ingress controller](https://grafana.com/grafana/dashboards/9614)

#### 参考资料

[徒手教你制作运维监控大屏](https://www.cnblogs.com/zhangs1986/p/11180694.html)

[Grafana全面瓦解](https://www.jianshu.com/p/7e7e0d06709b)

[grafana报错解决：Panel plugin not found grafana-piechart-panel](https://blog.csdn.net/xujiamin0022016/article/details/105291432/)

[Prometheus查询表达式](https://www.jianshu.com/p/d187ac561eb8)

[Prometheus 查询语言](https://www.jianshu.com/p/3bdc4cfa08da)

### 告警 alertmanager

[github](https://github.com/prometheus/alertmanager)

[CHANGELOG](https://github.com/prometheus/alertmanager/blob/master/CHANGELOG.md)

[hub官方](https://hub.docker.com/r/prom/alertmanager)

[Prometheus告警时机](https://blog.csdn.net/Jona_Li/article/details/105447755)

[Prometheus一条告警是怎么触发的](https://www.jianshu.com/p/af0f98fe7699)

[Prometheus AlertManager 实战](https://zhuanlan.zhihu.com/p/71922761)

[Prometheus 监控k8s告警rules](http://idcsec.com/2019/03/06/prometheus-%E7%9B%91%E6%8E%A7k8s%E5%91%8A%E8%AD%A6rules/)

```sh
docker run -d --name alertmanager -p 9093:9093 --restart=always \
  prom/alertmanager:v0.20.0

# 持久化数据
docker cp alertmanager:/etc/alertmanager /data/docker_volumn/

# 调整 alertmanager 配置
vi /data/docker_volumn/alertmanager/alertmanager.yml

docker stop alertmanager && docker rm -f alertmanager

docker run -d --name alertmanager -p 9093:9093 --hostname 10.250.15.39 --restart=always \
  -e TZ=Asia/Shanghai \
  -v /usr/share/zoneinfo/Asia/Shanghai:/etc/localtime:ro \
  -v /data/docker_volumn/alertmanager:/etc/alertmanager \
  prom/alertmanager:v0.20.0

# 调整 prometheus 配置
vi /data/docker_volumn/prometheus/prometheus.yml

docker restart prometheus
```

#### 告警规则配置

```sh
mkdir /data/docker_volumn/prometheus/rules

vi /data/docker_volumn/prometheus/rules/rule_xxx.yml

docker restart prometheus

# 重载 Prometheus 配置
curl -X POST http://localhost:9090/-/reload
```

### 钉钉告警

```sh
mkdir -p /data/docker_volumn/prometheus-webhook-dingtalk

vi /data/docker_volumn/prometheus-webhook-dingtalk/config.yml

docker stop prometheus-webhook-dingtalk && docker rm -f prometheus-webhook-dingtalk

docker run -d --name prometheus-webhook-dingtalk -p 8060:8060 --restart=always \
  -e TZ=Asia/Shanghai \
  -v /usr/share/zoneinfo/Asia/Shanghai:/etc/localtime:ro \
  -v /data/docker_volumn/prometheus-webhook-dingtalk/config.yml:/etc/prometheus-webhook-dingtalk/config.yml  \
  -v /data/docker_volumn/prometheus-webhook-dingtalk/default.tmpl:/etc/prometheus-webhook-dingtalk/templates/default.tmpl \
  timonwong/prometheus-webhook-dingtalk:v1.4.0 \
  --web.ui-enabled \
  --config.file=/etc/prometheus-webhook-dingtalk/config.yml
  
# 测试地址：

http://10.250.15.39:8060/ui/

curl 'https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxxxxxx' \
 -H 'Content-Type: application/json' \
 -d '{"msgtype": "text", 
      "text": {
           "content": "开发环境监控，我就是我，是不一样的烟火"
      }
    }'
```

#### 告警模版

在 promethues-webhook-dingtalk 配置 markdown 模版。

### 监控思路

[监控之我见](https://github.com/xiaoping378/blog/blob/master/posts/%E7%9B%91%E6%8E%A7%E4%B9%8B%E6%88%91%E8%A7%81.md#%E7%BD%91%E7%BB%9C%E7%9B%91%E6%8E%A7)

[IDC网络质量监控之（九）数据展示与告警篇(安装并调试prometheus_alertmanager+grafana+dingtlak)](https://boke.wsfnk.com/archives/905.html)

### 其他监控方案

[smokeping+prometheus+grafana](https://github.com/wilsonchai8/idc_ping_monitor)

[Prometheus + Clickhouse + Grafana 架构安装](https://www.jianshu.com/p/4f3c6bbbbfa9)

http://172.17.209.202:3000/
http://10.0.43.32:9100/
http://172.17.209.202:9090/

