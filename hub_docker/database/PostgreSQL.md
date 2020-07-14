# PostgreSQL

## 官方镜像

[Hub官方](https://hub.docker.com/_/postgres)

## 启动命令

```sh
# 持久化数据
docker volume create pgdata

docker stop postgres && docker rm postgres

docker run -d --name postgres -p 5432:5432 --restart always \
  -e POSTGRES_USER='root' \
  -e POSTGRES_PASSWORD='Scm2Postgres' \
  -e POSTGRES_DB='db_sonar' \
  -e ALLOW_IP_RANGE=0.0.0.0/0 \
  -v pgdata:/var/lib/postgresql/data \
  postgres:12.3
```