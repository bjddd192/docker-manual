# WordPress

WordPress 是使用 PHP 语言开发的博客平台，用户可以在支持 PHP 和 MySQL 数据库的服务器上架设属于自己的网站，也可以把 WordPress 当作一个内容管理系统（CMS）来使用。根据 W3techs 的统计，截至 2020 年 12 月，全球约 39.9% 的网站都使用 WordPress，无论是个人博客，还是官方网站，还是作为通用的内容管理系统，都可以通过 Wordpress 快速搭建，也是目前最流行的动态网站框架之一。

### 容器化部署

```yaml
version: '3.1'

services:

  wordpress:
    image: wordpress
    restart: always
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: 10.234.6.223
      WORDPRESS_DB_USER: exampleuser
      WORDPRESS_DB_PASSWORD: examplepass
      WORDPRESS_DB_NAME: exampledb
    volumes:
      - /data/docker_volumn/wordpress:/var/www/html
    network_mode: bridge

  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: exampledb
      MYSQL_USER: exampleuser
      MYSQL_PASSWORD: examplepass
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - /data/docker_volumn/mysql:/var/lib/mysql
    network_mode: bridge
```
