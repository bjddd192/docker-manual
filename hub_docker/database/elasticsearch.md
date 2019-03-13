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

## 参考资料

[Install Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)