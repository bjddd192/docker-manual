# elasticsearch

[官网](https://www.elastic.co/cn/)

[elastic past releases](https://www.elastic.co/cn/downloads/past-releases)

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

# 重启生效组件
docker restart kibana
```

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

## 参考资料

[ELKstack 中文指南](https://elkguide.elasticsearch.cn/logstash/)

[Install Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)

[elasticsearch 快速开始](https://www.cnblogs.com/cjsblog/p/9439331.html)

[Elasticsearch基本概念及核心配置文件详解](https://www.cnblogs.com/xiaochina/p/6855591.html)

[ELK的sentinl告警配置详解](https://www.cnblogs.com/amyzhu/p/10193557.html)

[elk报警监控之sentinl 钉钉+邮件告警](https://www.bbsmax.com/A/MyJxQAD25n/)

[kibana Sentinl插件](https://www.geeklive.cn/2019/04/01/kibana-sentinl/undefined/kibana-sentinl/)

[ES告警详解之Sentinl](https://www.tony-yin.site/2018/12/01/ES-Sentinl/)

