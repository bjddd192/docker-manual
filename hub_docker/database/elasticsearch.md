# elasticsearch

[官网](https://www.elastic.co/cn/)

[elastic past releases](https://www.elastic.co/cn/downloads/past-releases)

[Elastic开源与收费版本对比](https://www.elastic.co/cn/subscriptions)

kibana 告警解决方案：

[sirensolutions/sentinl](https://github.com/sirensolutions/sentinl)

[sentinl docs](https://sentinl.readthedocs.io/en/latest/)

## 官方镜像

[docker.elastic.co](https://www.docker.elastic.co/)

## 启动命令

```sh
docker pull docker.elastic.co/elasticsearch/elasticsearch:6.5.4

/sbin/sysctl -w vm.max_map_count=262144
  
docker stop elasticsearch && docker rm elasticsearch 

docker run -d --name elasticsearch --restart=always -p 9200:9200 -p 9300:9300 \
  --ulimit nofile=65536:65536 --ulimit memlock=-1:-1 \
  -e "bootstrap.memory_lock=true" \
  -e "discovery.type=single-node" \
  -e "ES_JAVA_OPTS=-Xms8g -Xmx8g" \
  docker.elastic.co/elasticsearch/elasticsearch:6.5.4

# 检查es健康状态：
curl http://127.0.0.1:9200/_cat/health
# 检查索引健康状态
curl http://127.0.0.1:9200/_cat/indices
```

## 持久化数据

```sh
docker stop elasticsearch && docker rm elasticsearch

docker run -d --name elasticsearch --restart=always -p 9200:9200 -p 9300:9300 \
--ulimit nofile=65536:65536 --ulimit memlock=-1:-1 \
-e "bootstrap.memory_lock=true" \
-e "discovery.type=single-node" \
-e "ES_JAVA_OPTS=-Xms8g -Xmx8g" \
docker.elastic.co/elasticsearch/elasticsearch:6.5.4

# 持久化数据
docker cp elasticsearch:/usr/share/elasticsearch /data/docker_volumn/

docker stop elasticsearch && docker rm elasticsearch

docker run -d --name elasticsearch --restart=always -p 9200:9200 -p 9300:9300 \
--ulimit nofile=65536:65536 --ulimit memlock=-1:-1 \
-e "bootstrap.memory_lock=true" \
-e "discovery.type=single-node" \
-e "ES_JAVA_OPTS=-Xms10g -Xmx10g" \
-v /data/docker_volumn/elasticsearch:/usr/share/elasticsearch \
harbor.bjds.belle.lan/k8s/elasticsearch:6.5.4

docker stop kibana && docker rm kibana

# docker 启动 kibana
docker run -d --name kibana -p 5601:5601 \
-e ELASTICSEARCH_URL=http://10.240.251.29:9200 \
harbor.bjds.belle.lan/k8s/kibana:6.5.4
```

[使用Docker搭建Elasticsearch 6.6.1集群](https://blog.csdn.net/weweeeeeeee/article/details/88405252)

## 查询表达式

```json
{
  "query": {
    "range": {
      "request_time": {
        "gt": 1
      }
    }
  }
}

http://elk-sls.belle.net.cn/app/kibana#/discover?_g=(refreshInterval:(display:Off,pause:!f,value:0),time:(from:now%2Fd,mode:quick,to:now%2Fd))&_a=(columns:!(message),filters:!(('$state':(store:appState),exists:(field:request_time),meta:(alias:!n,disabled:!f,index:'belle-net-cn-*',key:request_time,negate:!f,type:exists,value:exists)),('$state':(store:appState),meta:(alias:!n,disabled:!f,index:'belle-net-cn-*',key:host,negate:!f,type:phrase,value:hm-wms-rf.belle.net.cn),query:(match:(host:(query:hm-wms-rf.belle.net.cn,type:phrase)))),('$state':(store:appState),meta:(alias:!n,disabled:!f,index:'belle-net-cn-*',key:query,negate:!f,type:custom,value:'%7B%22range%22:%7B%22request_time%22:%7B%22gt%22:%222%22%7D%7D%7D'),query:(range:(request_time:(gt:'2'))))),index:'belle-net-cn-*',interval:auto,query:(match_all:()),sort:!('@timestamp',desc))
```

## kibana 告警配置

```sh
docker stop kibana && docker rm kibana

docker run -d --name kibana -p 5601:5601 \
-u root \
-e ELASTICSEARCH_URL=http://172.17.209.53:9200 \
hub.wonhigh.cn/k8s/kibana:6.5.4

# 持久化 kibana
docker cp kibana:/usr/share/kibana /data/docker_volumn/kibana

docker stop kibana && docker rm kibana

docker run -d --name kibana -p 5601:5601 \
-u root \
-e ELASTICSEARCH_URL=http://172.17.209.53:9200 \
-v /data/docker_volumn/kibana:/usr/share/kibana \
hub.wonhigh.cn/k8s/kibana:6.5.4

# 安装 sentinl 告警插件
docker exec -it kibana /usr/share/kibana/bin/kibana-plugin install http://10.0.43.24:8066/package/kibana/sentinl-v6.5.4.zip

# vi /data/docker_volumn/kibana/config/kibana.yml

# 重启生效组件
docker restart kibana
```

## kibana 告警配置(5.6.9版本)

```sh
docker stop kibana && docker rm kibana

docker run -d --name kibana -p 5601:5601 \
-u root \
-e ELASTICSEARCH_URL=http://10.250.11.105:9200 \
-e KIBANA_INDEX=.kibana-11.74 \
-e XPACK_GRAPH_ENABLED=false \
-e XPACK_MONITORING_ENABLED=false \
-e XPACK_REPORTING_ENABLED=false \
-e XPACK_SECURITY_ENABLED=false \
harbor.bjds.belle.lan/k8s/kibana:5.6.9 \
/bin/bash -c 'bin/kibana-plugin remove x-pack ; /usr/local/bin/kibana-docker'

# 持久化 kibana
docker cp kibana:/usr/share/kibana /data/docker_volumn/kibana

docker stop kibana && docker rm kibana

docker run -d --name kibana -p 5601:5601 \
-u root \
-e ELASTICSEARCH_URL=http://10.250.11.105:9200 \
-e KIBANA_INDEX=.kibana-11.74 \
-e XPACK_GRAPH_ENABLED=false \
-e XPACK_MONITORING_ENABLED=false \
-e XPACK_REPORTING_ENABLED=false \
-e XPACK_SECURITY_ENABLED=false \
-v /data/docker_volumn/kibana:/usr/share/kibana \
harbor.bjds.belle.lan/k8s/kibana:5.6.9 \
/bin/bash -c 'bin/kibana-plugin remove x-pack ; /usr/local/bin/kibana-docker'

# 安装 sentinl 告警插件
docker exec -it kibana /usr/share/kibana/bin/kibana-plugin install http://10.0.43.24:8066/package/kibana/sentinl-v5.6.9.zip

# 重启生效组件
docker restart kibana
```

### kibana连接需要认证的es

```sh
docker stop kibana && docker rm kibana

docker run -d --name kibana -p 5601:5601 \
-u root \
-e ELASTICSEARCH_URL=http://10.0.30.137:9200 \
kibana:7.6.2

# 持久化 kibana，更改配置文件
docker cp kibana:/usr/share/kibana /data/docker_volumn/kibana
chown 1000 -R /data/docker_volumn/kibana

docker stop kibana && docker rm kibana

docker run -d --name kibana -p 5601:5601 --restart=always \
-v /data/docker_volumn/kibana:/usr/share/kibana \
kibana:7.6.2
```

```yaml
# cat /data/docker_volumn/kibana/config/kibana.yml 
#
# ** THIS IS AN AUTO-GENERATED FILE **
#

# Default Kibana configuration for docker target
server.name: kibana
server.host: "0.0.0.0"
elasticsearch.hosts: [ "http://10.0.30.137:9200" ]
elasticsearch.requestTimeout: 60000
elasticsearch.username: "elastic"
elasticsearch.password: "belle123"
i18n.locale: "zh-CN"
```

[Docker 部署 kibana（ ES开启了密码认证）](https://www.cnblogs.com/evescn/p/14330193.html)

## 删除数据

[Delete By Query API](https://www.elastic.co/guide/en/elasticsearch/reference/6.5/docs-delete-by-query.html)

[Force merge API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-forcemerge.html#indices-forcemerge)

[elasticsearch Delete （根据条件删除）](https://www.cnblogs.com/zhuyeshen/p/10950560.html)

[ElasticSearch5.x 删除数据](https://cloud.tencent.com/developer/article/1350598)

[探究 | Elasticsearch如何物理删除给定期限的历史数据？](https://developer.aliyun.com/article/707400)

示例：

```sh
curl -X POST "http://172.17.209.53:9200/filebeat-*/_delete_by_query?conflicts=proceed" -H 'Content-Type: application/json' -d'
{
    "query": {
        "bool": {
            "must": [{
                "match_all": {}
            }, {
                "match_phrase": {
                    "kubernetes.container.name": {
                        "query": "dev-ept-api"
                    }
                }
            }, {
                "range": {
                    "@timestamp": {
                        "lt": "now-5d",
                        "format": "epoch_millis"
                    }
                }
            }]
        }
    }
}'

# curl -X POST "http://10.250.23.14:9200/filebeat-6.5.4-2020.03.17/_forcemerge?max_num_segments=1" 
```

若数据量太大删除很慢，且不能及时释放磁盘空间，强制合并耗时更是吓人，尽量不用此方式。

### Elastalert

[Yelp/elastalert Github](https://github.com/Yelp/elastalert)

[bitsensor/elastalert Hub docker](https://hub.docker.com/r/bitsensor/elastalert)

[ElastAlert 官方文档](https://elastalert.readthedocs.io/en/latest/)

[anjia0532/elastalert-docker Github](https://github.com/anjia0532/elastalert-docker)

[anjia0532/elastalert-docker Hub docker](https://hub.docker.com/r/anjia0532/elastalert-docker)

[Kibana Query Language](https://www.elastic.co/guide/en/kibana/current/kuery-query.html#kuery-query)

#### docker 部署

```sh
docker run -d --name elastalert -p 3030:3030 \
--net="host" \
bitsensor/elastalert:3.0.0-beta.0

# 持久化数据
docker cp elastalert:/opt/elastalert /data/docker_volumn/elastalert
docker cp elastalert:/opt/elastalert-server /data/docker_volumn/elastalert-server

# 修改配置文件
# /data/docker_volumn/elastalert-server/config/config.json
# /data/docker_volumn/elastalert/config.yaml

docker stop elastalert && docker rm -f elastalert

docker run -d --name elastalert -p 3030:3030 --restart=always \
-e "CONTAINER_TIMEZONE=Asia/Shanghai" \
-e "TZ=Asia/Shanghai" \
-v /data/docker_volumn/elastalert:/opt/elastalert \
-v /data/docker_volumn/elastalert-server:/opt/elastalert-server \
--net="host" \
--name elastalert bitsensor/elastalert:3.0.0-beta.0

docker logs -f elastalert
```

```sh
docker run -d --name elastalert -p 3030:3030 -p 3333:3333 \
--net="host" \
-e "CONTAINER_TIMEZONE=Asia/Shanghai" \
-e "TZ=Asia/Shanghai" \
-e "ELASTICSEARCH_HOST=172.17.209.53" \
anjia0532/elastalert-docker:v0.2.4

docker cp elastalert:/opt/elastalert /data/docker_volumn/elastalert

docker stop elastalert && docker rm -f elastalert

docker run -d --name elastalert -p 3030:3030 -p 3333:3333 \
--net="host" \
-e "CONTAINER_TIMEZONE=Asia/Shanghai" \
-e "TZ=Asia/Shanghai" \
-e "ELASTICSEARCH_HOST=172.17.209.53" \
-e "ELASTALERT_DINGTALK_ACCESS_TOKEN=67b1e7ec07cfed21eca6f4e7152a541e8d5b08a90c9ffa23b887f2b4f244b7f2" \
-e "ELASTALERT_DINGTALK_SECURITY_TYPE=sign" \
-e "ELASTALERT_DINGTALK_SECRET=SECd12d6a0181e1c844842be3280995454aa8130234e641d5a7b7f11ec5d7a839fe" \
-e "ELASTALERT_DINGTALK_AT_ALL=True" \
-e "ELASTALERT_RUN_EVERY=5" \
-e "ELASTALERT_BUFFER_TIME=15" \
-e "ELASTALERT_TIME_LIMIT=5" \
-v /data/docker_volumn/elastalert:/opt/elastalert \
anjia0532/elastalert-docker:v0.2.4

docker logs -f --tail 20 elastalert
```

### 异常处理

```sh
[2021-11-27T15:51:42,772][WARN ][o.e.d.s.f.s.h.UnifiedHighlighter] [nm-50lI] The length [10485760] of [message] field of [rycQYn0BnVG11kWS3KZq] doc of [filebeat-6.5.4-2021.11.27] index has exceeded the allowed maximum of [1000000] set for the next major Elastic version. This maximum can be set by changing the [index.highlight.max_analyzed_offset] index level setting. For large texts, indexing with offsets or term vectors is recommended!

# 索引默认偏移量设置过小，需要调整index.highlight.max_analyzed_offset的值，这个值默认为1000000，设置过大会给内存带来相当大负担，所以要根据服务器具体条件设置。
curl -XPUT http://127.0.0.1:9200/_settings -H 'Content-Type: application/json' -d' {"index" : {"highlight.max_analyzed_offset" : 100000000}}'
```

### 参考资料

[ELKstack 中文指南](https://elkguide.elasticsearch.cn/logstash/)

[Install Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)

[elasticsearch 快速开始](https://www.cnblogs.com/cjsblog/p/9439331.html)

[Elasticsearch基本概念及核心配置文件详解](https://www.cnblogs.com/xiaochina/p/6855591.html)

[ELK的sentinl告警配置详解](https://www.cnblogs.com/amyzhu/p/10193557.html)

[elk报警监控之sentinl 钉钉+邮件告警](https://www.bbsmax.com/A/MyJxQAD25n/)

[kibana Sentinl插件](https://www.geeklive.cn/2019/04/01/kibana-sentinl/undefined/kibana-sentinl/)

[ES告警详解之Sentinl](https://www.tony-yin.site/2018/12/01/ES-Sentinl/)

[ES告警详解之ElastAlert](https://www.tony-yin.site/2018/11/15/ES-ElastAlert/)

[Elasticsearch7.7告警模块Elastalert](https://www.fleyun.com/2020/09/10/elasticsearch7-7%E5%91%8A%E8%AD%A6%E6%A8%A1%E5%9D%97elastalert/)

[ElastAlert：『Hi，咱服务挂了』](https://blog.xizhibei.me/2017/11/19/alerting-with-elastalert/)

[ElastAlert监控日志告警Web攻击行为](https://www.freebuf.com/articles/web/160254.html)

[elastalert的简单运用](https://www.jianshu.com/p/f82812e0a743)

[有赞线上故障管理实践初探](https://tech.youzan.com/you-zan-xian-shang-gu-zhang-guan-li-shi-jian-chu-tan/)

[安装 Kibana（本地及 Docker）- Elastic Stack 实战手册](https://developer.aliyun.com/article/784123)

[Docker部署ELK（配置密码登录）及Elastalert企业微信告警配置](https://blog.csdn.net/xiaohai348257622/article/details/126707602)

[ELK7.11.2版本安装部署及ElastAlert告警相关配置](https://www.cnblogs.com/Oejfr/p/14585795.html)

[ElasticSearch Stack 各个版本收费情况](https://blog.csdn.net/vkingnew/article/details/91549698)

[ElastAlert-配置](https://blog.csdn.net/lihaipeng0417/article/details/118297163)
