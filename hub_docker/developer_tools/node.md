# node

### 构建脚本

```sh
docker run --rm -it -v /tmp/lesoon-integration-web:/user/app -w /user/app hub.wonhigh.cn/tools/node:10.24.1-alpine3.11 cnpm install

docker run --rm -it -v /tmp/lesoon-integration-web:/user/app -w /user/app hub.wonhigh.cn/tools/node:10.24.1-alpine3.11 cnpm run build
```

### 异常处理

[npm报错：npm WARN tar ENOENT: no such file or directory, open '*/node_modules/.staging/*'](https://blog.csdn.net/lgfx21/article/details/103319967)

[npm install返回syscall生成git错误(npm install returns syscall spawn git error)](https://www.it1352.com/1924515.html)

[node-sass加速](https://www.jianshu.com/p/c6a4f8048e57)

### 参考资料

[如何通过docker编译vue项目](https://segmentfault.com/a/1190000040260539?utm_source=tag-newest)

[The npm config files](https://www.npmjs.cn/files/npmrc/)
