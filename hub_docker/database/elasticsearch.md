# elasticsearch

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

## 参考资料

[Install Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)

[elasticsearch 快速开始](https://www.cnblogs.com/cjsblog/p/9439331.html)

[Elasticsearch基本概念及核心配置文件详解](https://www.cnblogs.com/xiaochina/p/6855591.html)
