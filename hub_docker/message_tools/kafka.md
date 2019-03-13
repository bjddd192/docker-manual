# kafka

docker run -d --name zookeeper -p 2181:2181 -t hub.wonhigh.cn/library/zookeeper:3.4.12
 
docker run  -d --name kafka \
-p 9092:9092 \
-e KAFKA_BROKER_ID=0 \
-e KAFKA_ZOOKEEPER_CONNECT=10.0.43.24:2181 \
-e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://10.0.43.24:9092 \
-e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -t wurstmeister/kafka
 
docker run -d --name kafka-manager \
--link zookeeper:zookeeper \
--link kafka:kafka \
-p 9000:9000 \
-e ZK_HOSTS=zookeeper:2181 sheepkiller/kafka-manager:1.3.1.8
