# Pinpoint

Generating DingTalk notification from Prometheus AlertManager WebHooks.

[官网](https://github.com/timonwong/prometheus-webhook-dingtalk/tree/v1.4.0)

[Hub官方](https://hub.docker.com/r/timonwong/prometheus-webhook-dingtalk)

## 容器化安装

```sh
mkdir -p /data/docker_volumn/prometheus-webhook-dingtalk

tee > /data/docker_volumn/prometheus-webhook-dingtalk/config.yml <<'EOF'
## Request timeout
# timeout: 5s

## Customizable templates path
# templates:
#   - contrib/templates/legacy/template.tmpl

## You can also override default template using `default_message`
## The following example to use the 'legacy' template from v0.3.0
# default_message:
#   title: '{{ template "legacy.title" . }}'
#   text: '{{ template "legacy.content" . }}'

## Targets, previously was known as "profiles"
targets:
  webhook1:
    url: https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxxxxxx
    # secret for signature
    secret: SECe90d9dd6c0555dabxxxxxx
  webhook2:
    url: https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxxxxxx
  webhook_legacy:
    url: https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxxxxxx
    # Customize template content
    message:
      # Use legacy template
      title: '{{ template "legacy.title" . }}'
      text: '{{ template "legacy.content" . }}'
  webhook_mention_all:
    url: https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxxxxxx
    mention:
      all: true
  webhook_mention_users:
    url: https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxxxxxx
    mention:
      mobiles: ['152xxxx7365', '186xxxx7105']
EOF

docker stop prometheus-webhook-dingtalk && docker rm -f prometheus-webhook-dingtalk

docker run -d --name prometheus-webhook-dingtalk -p 8060:8060 --restart=always \
  -v /data/docker_volumn/prometheus-webhook-dingtalk/config.yml:/etc/prometheus-webhook-dingtalk/config.yml  \
  hub.wonhigh.cn/k8s/prometheus-webhook-dingtalk:v1.4.0 \
  --web.ui-enabled \
  --config.file=/etc/prometheus-webhook-dingtalk/config.yml
```

测试地址：

http://10.0.43.31:8060/ui/

curl 'https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxxxxxx' \
   -H 'Content-Type: application/json' \
   -d '{"msgtype": "text", 
        "text": {
             "content": "开发环境监控，我就是我，是不一样的烟火"
        }
      }'


## 参考资料

[prometheus-operator自定义配置实践详解](https://blog.csdn.net/fanren224/article/details/73090082/)

[Prometheus 规则（rule）、模板配置说明](https://www.cnblogs.com/zhoujinyi/p/11947750.html)

[获取自定义机器人webhook](https://ding-doc.dingtalk.com/doc#/serverapi2/qf2nxq)

[使用钉钉完成对虚拟机监控告警](https://www.jianshu.com/p/302e1291e929)
