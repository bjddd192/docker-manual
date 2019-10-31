# dubbo-admin

## 官方镜像

[Hub官方](https://hub.docker.com/r/chenchuxin/dubbo-admin)

## 启动命令

```sh
docker stop dubbo-admin && docker rm dubbo-admin

docker run --name dubbo-admin -d -p 8080:8080 --restart=always \
    -e dubbo.registry.address=zookeeper://10.234.6.220:2181 \
    -e dubbo.admin.root.password=root \
    -e dubbo.admin.guest.password=guest \
    chenchuxin/dubbo-admin:latest
```

## Dockerfile

```sh
FROM maven:3.5-jdk-8-alpine AS build
LABEL Author="chenchuxin <idesireccx@gmail.com>"
WORKDIR /src
RUN apk add --no-cache git \
    && git clone https://github.com/apache/incubator-dubbo-ops \
    && cd incubator-dubbo-ops \
    && mvn package -Dmaven.test.skip=true

# timezone    
RUN apk add -U tzdata \
    && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

FROM tomcat:9-jre8-alpine
# timezone
COPY --from=build /etc/localtime /etc/localtime
WORKDIR /usr/local/tomcat/webapps
ARG version=2.0.0
RUN rm -rf ROOT
COPY --from=build /src/incubator-dubbo-ops/dubbo-admin/target/dubbo-admin-${version}.war .
RUN mv dubbo-admin-${version}.war ROOT.war
```
