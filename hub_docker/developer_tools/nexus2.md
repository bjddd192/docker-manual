# nexus2

## 官方镜像

[Hub官方](https://hub.docker.com/r/sonatype/nexus)

## 启动命令

```sh
mkdir /data/docker_volumn/nexus2/conf
mkdir /data/docker_volumn/nexus2/data

tee > /data/docker_volumn/nexus2/conf/nexus.properties <<'EOF'
# Jetty section
application-port=8081
application-host=0.0.0.0
nexus-webapp=${bundleBasedir}/nexus
nexus-webapp-context-path=/nexus
# Nexus section
nexus-work=${bundleBasedir}/../sonatype-work/nexus
runtime=${bundleBasedir}/nexus/WEB-INF
org.sonatype.nexus.proxy.maven.routing.Config.prefixFileMaxSize=500000
EOF

chown -R 200:200 /data/docker_volumn/nexus2

docker stop nexus && docker rm nexus

docker run --name nexus -d -p 20020:8081 --restart=always \
  -e TZ=Asia/Shanghai \
  -e CONTEXT_PATH=/nexus \
  -e MAX_HEAP=768m \
  -e MIN_HEAP=256m \
  -v /data/docker_volumn/nexus2/data:/sonatype-work \
  -v /data/docker_volumn/nexus2/conf/nexus.properties:/opt/sonatype/nexus/conf/nexus.properties \
  sonatype/nexus:2.14.11
```

## 验证

访问 url 如：http://serverIP:20020/nexus，初始用户名/密码：admin/admin123