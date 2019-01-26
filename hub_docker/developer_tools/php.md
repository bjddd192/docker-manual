# php

## 官方镜像

[Hub官方](https://hub.docker.com/_/php)

## 测试文件

hello.php

```php
<html>
 <head>
  <title>PHP 测试</title>
 </head>
 <body>
 <?php echo '<p>Hello World</p>'; ?>
 </body>
</html>
```

## 启动命令

```sh
docker stop php-app && docker rm php-app

docker run --name php-app -d -p 30028:80 --restart=always \
    -v /home/docker/php:/var/www/html \
    php:7.2-apache
```

## 验证

访问 url 如：http://203.195.243.110:30028/hello.php
