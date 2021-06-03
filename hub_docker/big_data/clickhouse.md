# clickhouse

[官网](https://clickhouse.tech/)

[官方文档](https://clickhouse.tech/docs/v20.3/zh/)

[Yandex Cloud](https://console.cloud.yandex.com/)

[中国社区论坛](http://www.clickhouse.com.cn/)

### 官方镜像

[yandex/clickhouse-server](https://hub.docker.com/r/yandex/clickhouse-server/)

[yandex/clickhouse-client](https://hub.docker.com/r/yandex/clickhouse-client)

### 官方在线练习

[Clickhouse Playground](https://play.clickhouse.tech/?file=playground)

### 安装部署

[ClickHouse国家级项目最佳实践](https://www.jiqizhixin.com/articles/2020-05-06-5)

[Clickhouse 在腾讯的应用实践](https://www.jiqizhixin.com/articles/2019-10-25-3)

[clickhouse搭建](http://blog.sunqiang.me/2020/01/09/clickhouse%E6%90%AD%E5%BB%BA/)

#### 离线包下载

[官方下载](https://repo.yandex.ru/clickhouse/rpm/)

```sh
wget https://repo.yandex.ru/clickhouse/rpm/lts/x86_64/clickhouse-client-20.8.7.15-2.noarch.rpm
wget https://repo.yandex.ru/clickhouse/rpm/lts/x86_64/clickhouse-common-static-20.8.7.15-2.x86_64.rpm
wget https://repo.yandex.ru/clickhouse/rpm/lts/x86_64/clickhouse-server-20.8.7.15-2.noarch.rpm
wget https://repo.yandex.ru/clickhouse/rpm/lts/x86_64/clickhouse-common-static-dbg-20.8.7.15-2.x86_64.rpm
wget https://repo.yandex.ru/clickhouse/rpm/lts/x86_64/clickhouse-test-20.8.7.15-2.noarch.rpm
```

#### CentOS yum 部署

```sh
yum -y install yum-utils
yum-config-manager --add-repo https://repo.clickhouse.tech/rpm/clickhouse.repo
yum makecache fast
yum list clickhouse-server --showduplicates | sort -r
rpm --import https://repo.clickhouse.tech/CLICKHOUSE-KEY.GPG
yum -y install clickhouse-server-20.8.7.15-2
yum -y install clickhouse-server-20.8.7.15-2
yum list installed 'clickhouse*'

systemctl start clickhouse-server 
systemctl status clickhouse-server

netstat -nltp | ag 8123


# 卸载Clickhouse
systemctl stop clickhouse-server
yum list installed | grep clickhouse
yum remove -y clickhouse-server
yum remove -y clickhouse-common-static
rm -rf /var/lib/clickhouse
rm -rf /etc/clickhouse-*
rm -rf /var/log/clickhouse-server
rm -rf /data/clickhouse
```

#### CentOS 离线部署

```sh
# 查看CPU是否支持 SSE 4.2
grep -q sse4_2 /proc/cpuinfo && echo "SSE 4.2 supported" || echo "SSE 4.2 not supported"

```

#### Docker部署

```sh
docker pull yandex/clickhouse-server:20.8.7.15
docker pull yandex/clickhouse-client:20.8.7.15
  
docker stop clickhouse-server && docker rm clickhouse-server 

# 端口说明：
# tcp_port   9000
# http_port  8123
# mysql_port 9004
# interserver_http_port(同步端口) 9009
docker run -d --name clickhouse-server -p 9000:9000 -p 8123:8123 -p 9004:9004 -p 9009:9009 --restart=always \
  --ulimit nofile=262144:262144 \
  -v /data/docker_volumn/clickhouse-server:/var/lib/clickhouse \
  yandex/clickhouse-server:20.8.7.15

# 持久化
# 数据目录：/var/lib/clickhouse
# 配置目录：/etc/clickhouse-server
# 日志目录：/var/log/clickhouse-server
docker cp clickhouse-server:/etc/clickhouse-server/config.xml /data/docker_volumn/clickhouse-server/config.xml

# 验证：
curl http://localhost:8123

# 从本地客户端连接Server
docker run -it --rm --link clickhouse-server:clickhouse-server \
  yandex/clickhouse-client:20.8.7.15 --host clickhouse-server
:) SHOW DATABASES;

SHOW DATABASES

┌─name────┐
│ default │
│ system  │
└─────────┘

# 容器内安装工具软件
tee /etc/apt/sources.list <<-'EOF'
# ubuntu18.04(bionic) 配置阿里数据源

deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
EOF

apt-get update
apt-get install -y net-tools inetutils-ping telnet
```

#### docker-compoese集群部署

2 分片 2 副本

```sh
# 1、部署zk集群(3台)
docker-compose up -d zoo1 zoo2 zoo3
# 2、拷贝一份单机版本的配置文件，在此基础上进行调整为集群配置
docker-compose up -d chs01011 chs01012 chs01021 chs01022

# 查看集群状态
SELECT *
FROM system.clusters;

┌─cluster───────┬─shard_num─┬─shard_weight─┬─replica_num─┬─host_name─┬─host_address──┬─port─┬─is_local─┬
│ cluster_chs01 │         1 │            1 │           1 │ chs01011  │ 192.168.150.5 │ 9000 │        0 │
│ cluster_chs01 │         1 │            1 │           2 │ chs01012  │ 192.168.150.7 │ 9000 │        0 │
│ cluster_chs01 │         2 │            1 │           1 │ chs01021  │ 192.168.150.6 │ 9000 │        0 │
│ cluster_chs01 │         2 │            1 │           2 │ chs01022  │ 192.168.150.8 │ 9000 │        1 │
└───────────────┴───────────┴──────────────┴─────────────┴───────────┴───────────────┴──────┴──────────┴
```

##### 参考资料

[jneo8 / clickhouse-setup](https://github.com/jneo8/clickhouse-setup)

[clickhouse-cluster-example](https://github.com/hughshen/clickhouse-cluster-example)

#### k8s部署

[ xiaods / k8s-clickhouse-v2 ](https://github.com/xiaods/k8s-clickhouse-v2)

#### 访问权限控制

```sh
# 密码可以明文

# 密码加密方法有两种：
# generate decent password
PASSWORD=$(base64 < /dev/urandom | head -c8); echo "$PASSWORD"; echo -n "$PASSWORD" | sha256sum | tr -d '-'
# generate double SHA1
PASSWORD=$(base64 < /dev/urandom | head -c8); echo "$PASSWORD"; echo -n "$PASSWORD" | sha1sum | tr -d '-' | xxd -r -p | sha1sum | tr -d '-'
```

### 客户端连接工具

[第三方开发的可视化界面](https://clickhouse.tech/docs/zh/interfaces/third-party/gui/)

[HTTP 客户端](https://clickhouse.tech/docs/v20.3/zh/interfaces/http/)

#### 命令行参数

```sh
--host, -h -– 服务端的 host 名称, 默认是 'localhost'。 您可以选择使用 host 名称或者 IPv4 或 IPv6 地址。
--port – 连接的端口，默认值： 9000。注意 HTTP 接口以及 TCP 原生接口是使用不同端口的。
--user, -u – 用户名。 默认值： default。
--password – 密码。 默认值： 空字符串。
--query, -q – 非交互模式下的查询语句.
--database, -d – 默认当前操作的数据库. 默认值： 服务端默认的配置 （默认是 default）。
--multiline, -m – 如果指定，允许多行语句查询（Enter 仅代表换行，不代表查询语句完结）。
--multiquery, -n – 如果指定, 允许处理用逗号分隔的多个查询，只在非交互模式下生效。
--format, -f – 使用指定的默认格式输出结果。
--vertical, -E – 如果指定，默认情况下使用垂直格式输出结果。这与 '--format=Vertical' 相同。在这种格式中，每个值都在单独的行上打印，这种方式对显示宽表很有帮助。
--time, -t – 如果指定，非交互模式下会打印查询执行的时间到 'stderr' 中。
--stacktrace – 如果指定，如果出现异常，会打印堆栈跟踪信息。
--config-file – 配置文件的名称。
```

#### DBeaver

[官网下载](https://dbeaver.io/download/)



### ClickHouse学习

ClickHouse是一个用于联机分析(OLAP)的列式数据库管理系统(DBMS)。

对于存储而言，列式数据库总是将同一列的数据存储在一起，不同列的数据也总是分开存储。

不同的存储方式适合不同的场景，这里的查询场景包括： 进行了哪些查询，多久查询一次以及各类查询的比例； 每种查询读取多少数据————行、列和字节；读取数据和写入数据之间的关系；使用的数据集大小以及如何使用本地的数据集；是否使用事务,以及它们是如何进行隔离的；数据的复制机制与数据的完整性要求；每种类型的查询要求的延迟与吞吐量等等。

吞吐量可以使用每秒处理的行数或每秒处理的字节数来衡量。

单个服务器上建议每秒最多查询100次。

ClickHouse原理解析与应用实践

#### ClickHouse的独特功能

- 真正的列式数据库管理系统

- 数据压缩

- 数据的磁盘存储

- 多核心并行处理

- 多服务器分布式处理

- 支持SQL

- 向量引擎

- 实时的数据更新

- 索引

- 适合在线查询

- 支持近似计算

- 支持数据复制和数据完整性

#### ClickHouse缺乏的功能

1. 没有完整的事务支持。
2. 缺少高频率，低延迟的修改或删除已存在数据的能力。仅能用于批量删除或修改数据，但这符合 [GDPR](https://gdpr-info.eu/)。
3. 稀疏索引使得ClickHouse不适合通过其键检索单行的点查询。

#### 下载学习数据

```sh
curl https://clickhouse-datasets.s3.yandex.net/hits/tsv/hits_v1.tsv.xz | unxz --threads=`nproc` > hits_v1.tsv
curl https://clickhouse-datasets.s3.yandex.net/visits/tsv/visits_v1.tsv.xz | unxz --threads=`nproc` > visits_v1.tsv
# hits 是一个表格，其中所有用户在服务覆盖的所有网站上执行的所有操作。
# visits 是包含预建会话而不是单个操作的表。

# 解压数据包
unxz -k hits_v1.tsv.xz
unxz -k visits_v1.tsv.xz

# 创建数据库、表

# 导入数据
cat hits_v1.tsv | clickhouse-client --query "INSERT INTO tutorial.hits_v1 FORMAT TSV" --max_insert_block_size=100000
cat visits_v1.tsv | clickhouse-client --query "INSERT INTO tutorial.visits_v1 FORMAT TSV" --max_insert_block_size=100000

# 查询数据
clickhouse-client --query "OPTIMIZE TABLE tutorial.hits_v1 FINAL"
clickhouse-client --query "SELECT COUNT(*) FROM tutorial.hits_v1"
clickhouse-client --query "OPTIMIZE TABLE tutorial.visits_v1 FINAL"
clickhouse-client --query "SELECT COUNT(*) FROM tutorial.visits_v1"
```



#### 数据库操作

```mysql
docker exec -it --env COLUMNS=200 --env LINES=200 clickhouse-server bash

-- 创建数据库
clickhouse-client -q "show databases"
clickhouse-client -q "CREATE DATABASE IF NOT EXISTS tutorial"
-- 创建一个mysql引擎数据库(相当于dblink)
clickhouse-client -q "CREATE DATABASE IF NOT EXISTS db_test ENGINE = MySQL('10.0.30.39:3306', 'test', 'admin', 'Admttooerif0')"


CREATE TABLE tutorial.hits_v1
(
    `WatchID` UInt64,
    `JavaEnable` UInt8,
    `Title` String,
    `GoodEvent` Int16,
    `EventTime` DateTime,
    `EventDate` Date,
    `CounterID` UInt32,
    `ClientIP` UInt32,
    `ClientIP6` FixedString(16),
    `RegionID` UInt32,
    `UserID` UInt64,
    `CounterClass` Int8,
    `OS` UInt8,
    `UserAgent` UInt8,
    `URL` String,
    `Referer` String,
    `URLDomain` String,
    `RefererDomain` String,
    `Refresh` UInt8,
    `IsRobot` UInt8,
    `RefererCategories` Array(UInt16),
    `URLCategories` Array(UInt16),
    `URLRegions` Array(UInt32),
    `RefererRegions` Array(UInt32),
    `ResolutionWidth` UInt16,
    `ResolutionHeight` UInt16,
    `ResolutionDepth` UInt8,
    `FlashMajor` UInt8,
    `FlashMinor` UInt8,
    `FlashMinor2` String,
    `NetMajor` UInt8,
    `NetMinor` UInt8,
    `UserAgentMajor` UInt16,
    `UserAgentMinor` FixedString(2),
    `CookieEnable` UInt8,
    `JavascriptEnable` UInt8,
    `IsMobile` UInt8,
    `MobilePhone` UInt8,
    `MobilePhoneModel` String,
    `Params` String,
    `IPNetworkID` UInt32,
    `TraficSourceID` Int8,
    `SearchEngineID` UInt16,
    `SearchPhrase` String,
    `AdvEngineID` UInt8,
    `IsArtifical` UInt8,
    `WindowClientWidth` UInt16,
    `WindowClientHeight` UInt16,
    `ClientTimeZone` Int16,
    `ClientEventTime` DateTime,
    `SilverlightVersion1` UInt8,
    `SilverlightVersion2` UInt8,
    `SilverlightVersion3` UInt32,
    `SilverlightVersion4` UInt16,
    `PageCharset` String,
    `CodeVersion` UInt32,
    `IsLink` UInt8,
    `IsDownload` UInt8,
    `IsNotBounce` UInt8,
    `FUniqID` UInt64,
    `HID` UInt32,
    `IsOldCounter` UInt8,
    `IsEvent` UInt8,
    `IsParameter` UInt8,
    `DontCountHits` UInt8,
    `WithHash` UInt8,
    `HitColor` FixedString(1),
    `UTCEventTime` DateTime,
    `Age` UInt8,
    `Sex` UInt8,
    `Income` UInt8,
    `Interests` UInt16,
    `Robotness` UInt8,
    `GeneralInterests` Array(UInt16),
    `RemoteIP` UInt32,
    `RemoteIP6` FixedString(16),
    `WindowName` Int32,
    `OpenerName` Int32,
    `HistoryLength` Int16,
    `BrowserLanguage` FixedString(2),
    `BrowserCountry` FixedString(2),
    `SocialNetwork` String,
    `SocialAction` String,
    `HTTPError` UInt16,
    `SendTiming` Int32,
    `DNSTiming` Int32,
    `ConnectTiming` Int32,
    `ResponseStartTiming` Int32,
    `ResponseEndTiming` Int32,
    `FetchTiming` Int32,
    `RedirectTiming` Int32,
    `DOMInteractiveTiming` Int32,
    `DOMContentLoadedTiming` Int32,
    `DOMCompleteTiming` Int32,
    `LoadEventStartTiming` Int32,
    `LoadEventEndTiming` Int32,
    `NSToDOMContentLoadedTiming` Int32,
    `FirstPaintTiming` Int32,
    `RedirectCount` Int8,
    `SocialSourceNetworkID` UInt8,
    `SocialSourcePage` String,
    `ParamPrice` Int64,
    `ParamOrderID` String,
    `ParamCurrency` FixedString(3),
    `ParamCurrencyID` UInt16,
    `GoalsReached` Array(UInt32),
    `OpenstatServiceName` String,
    `OpenstatCampaignID` String,
    `OpenstatAdID` String,
    `OpenstatSourceID` String,
    `UTMSource` String,
    `UTMMedium` String,
    `UTMCampaign` String,
    `UTMContent` String,
    `UTMTerm` String,
    `FromTag` String,
    `HasGCLID` UInt8,
    `RefererHash` UInt64,
    `URLHash` UInt64,
    `CLID` UInt32,
    `YCLID` UInt64,
    `ShareService` String,
    `ShareURL` String,
    `ShareTitle` String,
    `ParsedParams` Nested(
        Key1 String,
        Key2 String,
        Key3 String,
        Key4 String,
        Key5 String,
        ValueDouble Float64),
    `IslandID` FixedString(16),
    `RequestNum` UInt32,
    `RequestTry` UInt8
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(EventDate)
ORDER BY (CounterID, EventDate, intHash32(UserID))
SAMPLE BY intHash32(UserID)
SETTINGS index_granularity = 8192;

CREATE TABLE tutorial.visits_v1
(
    `CounterID` UInt32,
    `StartDate` Date,
    `Sign` Int8,
    `IsNew` UInt8,
    `VisitID` UInt64,
    `UserID` UInt64,
    `StartTime` DateTime,
    `Duration` UInt32,
    `UTCStartTime` DateTime,
    `PageViews` Int32,
    `Hits` Int32,
    `IsBounce` UInt8,
    `Referer` String,
    `StartURL` String,
    `RefererDomain` String,
    `StartURLDomain` String,
    `EndURL` String,
    `LinkURL` String,
    `IsDownload` UInt8,
    `TraficSourceID` Int8,
    `SearchEngineID` UInt16,
    `SearchPhrase` String,
    `AdvEngineID` UInt8,
    `PlaceID` Int32,
    `RefererCategories` Array(UInt16),
    `URLCategories` Array(UInt16),
    `URLRegions` Array(UInt32),
    `RefererRegions` Array(UInt32),
    `IsYandex` UInt8,
    `GoalReachesDepth` Int32,
    `GoalReachesURL` Int32,
    `GoalReachesAny` Int32,
    `SocialSourceNetworkID` UInt8,
    `SocialSourcePage` String,
    `MobilePhoneModel` String,
    `ClientEventTime` DateTime,
    `RegionID` UInt32,
    `ClientIP` UInt32,
    `ClientIP6` FixedString(16),
    `RemoteIP` UInt32,
    `RemoteIP6` FixedString(16),
    `IPNetworkID` UInt32,
    `SilverlightVersion3` UInt32,
    `CodeVersion` UInt32,
    `ResolutionWidth` UInt16,
    `ResolutionHeight` UInt16,
    `UserAgentMajor` UInt16,
    `UserAgentMinor` UInt16,
    `WindowClientWidth` UInt16,
    `WindowClientHeight` UInt16,
    `SilverlightVersion2` UInt8,
    `SilverlightVersion4` UInt16,
    `FlashVersion3` UInt16,
    `FlashVersion4` UInt16,
    `ClientTimeZone` Int16,
    `OS` UInt8,
    `UserAgent` UInt8,
    `ResolutionDepth` UInt8,
    `FlashMajor` UInt8,
    `FlashMinor` UInt8,
    `NetMajor` UInt8,
    `NetMinor` UInt8,
    `MobilePhone` UInt8,
    `SilverlightVersion1` UInt8,
    `Age` UInt8,
    `Sex` UInt8,
    `Income` UInt8,
    `JavaEnable` UInt8,
    `CookieEnable` UInt8,
    `JavascriptEnable` UInt8,
    `IsMobile` UInt8,
    `BrowserLanguage` UInt16,
    `BrowserCountry` UInt16,
    `Interests` UInt16,
    `Robotness` UInt8,
    `GeneralInterests` Array(UInt16),
    `Params` Array(String),
    `Goals` Nested(
        ID UInt32,
        Serial UInt32,
        EventTime DateTime,
        Price Int64,
        OrderID String,
        CurrencyID UInt32),
    `WatchIDs` Array(UInt64),
    `ParamSumPrice` Int64,
    `ParamCurrency` FixedString(3),
    `ParamCurrencyID` UInt16,
    `ClickLogID` UInt64,
    `ClickEventID` Int32,
    `ClickGoodEvent` Int32,
    `ClickEventTime` DateTime,
    `ClickPriorityID` Int32,
    `ClickPhraseID` Int32,
    `ClickPageID` Int32,
    `ClickPlaceID` Int32,
    `ClickTypeID` Int32,
    `ClickResourceID` Int32,
    `ClickCost` UInt32,
    `ClickClientIP` UInt32,
    `ClickDomainID` UInt32,
    `ClickURL` String,
    `ClickAttempt` UInt8,
    `ClickOrderID` UInt32,
    `ClickBannerID` UInt32,
    `ClickMarketCategoryID` UInt32,
    `ClickMarketPP` UInt32,
    `ClickMarketCategoryName` String,
    `ClickMarketPPName` String,
    `ClickAWAPSCampaignName` String,
    `ClickPageName` String,
    `ClickTargetType` UInt16,
    `ClickTargetPhraseID` UInt64,
    `ClickContextType` UInt8,
    `ClickSelectType` Int8,
    `ClickOptions` String,
    `ClickGroupBannerID` Int32,
    `OpenstatServiceName` String,
    `OpenstatCampaignID` String,
    `OpenstatAdID` String,
    `OpenstatSourceID` String,
    `UTMSource` String,
    `UTMMedium` String,
    `UTMCampaign` String,
    `UTMContent` String,
    `UTMTerm` String,
    `FromTag` String,
    `HasGCLID` UInt8,
    `FirstVisit` DateTime,
    `PredLastVisit` Date,
    `LastVisit` Date,
    `TotalVisits` UInt32,
    `TraficSource` Nested(
        ID Int8,
        SearchEngineID UInt16,
        AdvEngineID UInt8,
        PlaceID UInt16,
        SocialSourceNetworkID UInt8,
        Domain String,
        SearchPhrase String,
        SocialSourcePage String),
    `Attendance` FixedString(16),
    `CLID` UInt32,
    `YCLID` UInt64,
    `NormalizedRefererHash` UInt64,
    `SearchPhraseHash` UInt64,
    `RefererDomainHash` UInt64,
    `NormalizedStartURLHash` UInt64,
    `StartURLDomainHash` UInt64,
    `NormalizedEndURLHash` UInt64,
    `TopLevelDomain` UInt64,
    `URLScheme` UInt64,
    `OpenstatServiceNameHash` UInt64,
    `OpenstatCampaignIDHash` UInt64,
    `OpenstatAdIDHash` UInt64,
    `OpenstatSourceIDHash` UInt64,
    `UTMSourceHash` UInt64,
    `UTMMediumHash` UInt64,
    `UTMCampaignHash` UInt64,
    `UTMContentHash` UInt64,
    `UTMTermHash` UInt64,
    `FromHash` UInt64,
    `WebVisorEnabled` UInt8,
    `WebVisorActivity` UInt32,
    `ParsedParams` Nested(
        Key1 String,
        Key2 String,
        Key3 String,
        Key4 String,
        Key5 String,
        ValueDouble Float64),
    `Market` Nested(
        Type UInt8,
        GoalID UInt32,
        OrderID String,
        OrderPrice Int64,
        PP UInt32,
        DirectPlaceID UInt32,
        DirectOrderID UInt32,
        DirectBannerID UInt32,
        GoodID String,
        GoodName String,
        GoodQuantity Int32,
        GoodPrice Int64),
    `IslandID` FixedString(16)
)
ENGINE = CollapsingMergeTree(Sign)
PARTITION BY toYYYYMM(StartDate)
ORDER BY (CounterID, StartDate, intHash32(UserID), VisitID)
SAMPLE BY intHash32(UserID)
SETTINGS index_granularity = 8192;

-- 查询系统设置
SELECT name, value, changed, description
FROM system.settings
WHERE name LIKE '%max_insert_b%'
FORMAT TSV;

-- 查看字符长度(1个汉字3字节)
SELECT length('啊阳磊123')
┌─length('啊阳磊123')─┐
│                  12 │
└─────────────────────┘
SELECT lengthUTF8('啊阳磊123')
┌─lengthUTF8('啊阳磊123')─┐
│                       6 │
└─────────────────────────┘

-- 生成UUID
SELECT generateUUIDv4()

-- 查看当前时间
SELECT now() as current_date_time, current_date_time + INTERVAL 4 DAY
```

#### ClickHouse原理解析与应用实践

[演示代码与样例数据](https://github.com/nauu/clickhousebook)

### 表引擎

|  引擎类型   | 特点                                                         |
| :---------: | ------------------------------------------------------------ |
|  MergeTree  | 适用于高负载任务的最通用和功能最强大的表引擎。这些引擎的共同特点是可以快速插入数据并进行后续的后台数据处理。 MergeTree系列引擎支持数据复制，分区和一些其他引擎不支持的其他功能。<br />存储的数据按主键排序。<br />允许使用分区。<br />支持数据副本。<br />支持数据采样。 |
|     Log     | 具有最小功能的[轻量级引擎](https://clickhouse.tech/docs/v20.3/zh/operations/table_engines/log_family/)。当您需要快速写入许多小表（最多约100万行）并在以后整体读取它们时，该类型的引擎是最有效的。 |
| Intergation | 用于与其他的数据存储与处理系统集成的引擎。                   |
|             |                                                              |

#### 文件引擎

```sh
tee /var/lib/clickhouse/user_files/test.csv <<-'EOF'
1,2,3
3,2,1
78,43,45
EOF

clickhouse-client -m -q "
SELECT *
FROM file('/var/lib/clickhouse/user_files/test.csv', 'CSV', 'c1 UInt32, c2 UInt32, c3 UInt32')
LIMIT 2;"
```

### DDL

#### 数据库定义

```mysql
-- 创建一个默认引擎数据库
CREATE DATABASE IF NOT EXISTS tutorial;

-- 创建一个mysql引擎数据库(相当于dblink)
CREATE DATABASE IF NOT EXISTS db_test ENGINE = MySQL('10.0.30.39:3306', 'test', 'admin', 'Admttooerif0');
```

#### 基础表定义

```mysql
CREATE table tutorial.bm_size as db_test.bm_size;

-- 快速复制一个表
CREATE table IF NOT EXISTS tutorial.bm_size as db_test.bm_size ENGINE = Memory;

-- 快速复制一个表同时复制数据
CREATE table IF NOT EXISTS tutorial.bm_store ENGINE = Memory as select * FROM db_test.bm_size;

-- 查看建表语句
show CREATE table tutorial.bm_size;

-- 查看表结构
desc bm_size;

-- 表结构定义(Memory引擎)
-- 尺码资料
-- DROP TABLE tutorial.bm_size;
CREATE TABLE IF NOT EXISTS tutorial.bm_size (
 `id` Int64 COMMENT '尺码资料-主键ID',
 `company_id` Int64 COMMENT '租赁公司ID(SAAS预留字段)',
 `size_no` String COMMENT '尺码编号',
 `size_code` Nullable(String) COMMENT '尺码编码',
 `size_name` String COMMENT '尺码名称',
 `owner_no` String DEFAULT 'BL' COMMENT '货主编号',
 `creator` String COMMENT '创建人',
 `create_time` DateTime COMMENT '创建时间',
 `modifier` Nullable(String) COMMENT '修改人',
 `modify_time` Nullable(DateTime) COMMENT '修改时间',
 `remarks` Nullable(String) COMMENT '备注',
 `update_time` DateTime COMMENT '记录更新时间'
) ENGINE = Memory;

-- 尺码资料(MergeTree引擎)
-- DROP TABLE tutorial.bm_size;
CREATE TABLE IF NOT EXISTS tutorial.bm_size (
 `id` Int64 COMMENT '尺码资料-主键ID',
 `company_id` Int64 COMMENT '租赁公司ID(SAAS预留字段)',
 `size_no` String COMMENT '尺码编号',
 `size_code` Nullable(String) COMMENT '尺码编码',
 `size_name` String COMMENT '尺码名称',
 `owner_no` String DEFAULT 'BL' COMMENT '货主编号',
 `creator` String COMMENT '创建人',
 `create_time` DateTime COMMENT '创建时间',
 `modifier` Nullable(String) COMMENT '修改人',
 `modify_time` Nullable(DateTime) COMMENT '修改时间',
 `remarks` Nullable(String) COMMENT '备注',
 `update_time` DateTime COMMENT '记录更新时间'
) ENGINE = MergeTree()
PRIMARY KEY size_no
ORDER BY size_no;

-- 增加列(需MergeTree、Merge、Distributed表引擎才支持)
ALTER TABLE tutorial.bm_size add column if not exists `order_no` String DEFAULT '0' COMMENT '序号' AFTER owner_no;
-- 增加列(需MergeTree、Merge、Distributed表引擎才支持)
ALTER TABLE tutorial.bm_size modify column if exists  `order_no` Int32 DEFAULT 0 COMMENT '序号';

-- 删除列(需MergeTree、Merge、Distributed表引擎才支持)
ALTER TABLE tutorial.bm_size drop column if exists `order_no`;

-- 移动表、重命名表(只能在单数据库节点中移动)
RENAME TABLE tutorial.bm_size TO tutorial.bm_size2;
RENAME TABLE tutorial.bm_size2 TO tutorial.bm_size;

-- 清空表
TRUNCATE TABLE if exists tutorial.bm_size2;

-- 删除表
DROP TABLE tutorial.bm_size;
```

#### 临时表定义

临时表只在会话范围有效，不属于任何数据库，主要用于集群间数据传播。

```mysql
-- 尺码资料临时表
-- DROP TABLE IF EXISTS tmp_bm_size;
CREATE TEMPORARY TABLE IF NOT EXISTS tmp_bm_size (
 `id` Int64 COMMENT '尺码资料临时表-主键ID',
 `company_id` Int64 COMMENT '租赁公司ID(SAAS预留字段)',
 `size_no` String COMMENT '尺码编号',
 `size_code` Nullable(String) COMMENT '尺码编码',
 `size_name` String COMMENT '尺码名称',
 `owner_no` String DEFAULT 'BL' COMMENT '货主编号',
 `creator` String COMMENT '创建人',
 `create_time` DateTime COMMENT '创建时间',
 `modifier` Nullable(String) COMMENT '修改人',
 `modify_time` Nullable(DateTime) COMMENT '修改时间',
 `remarks` Nullable(String) COMMENT '备注',
 `update_time` DateTime COMMENT '记录更新时间'
) ENGINE = Memory;
```

#### 视图定义

```mysql
-- 普通视图
-- DROP TABLE IF EXISTS tutorial.v_bm_size;
CREATE VIEW IF NOT EXISTS tutorial.v_bm_size
AS
SELECT * FROM tutorial.bm_size;

-- 物化视图(不初始化数据)
-- DROP TABLE IF EXISTS tutorial.mv_bm_size;
CREATE MATERIALIZED VIEW IF NOT EXISTS tutorial.mv_bm_size
ENGINE = Memory
AS
SELECT * FROM tutorial.bm_size;

-- 物化视图(初始化数据)
-- DROP TABLE IF EXISTS tutorial.mv_bm_size;
CREATE MATERIALIZED VIEW IF NOT EXISTS tutorial.mv_bm_size
ENGINE = Memory
POPULATE
AS
SELECT * FROM tutorial.bm_size;

-- 物化视图建立后，如源表有新数据插入，则物化视图会同步更新，但是不支持同步删除。
-- 物化视图是使用了 .inner 特殊前缀的数据表。
```

#### 分区表定义

```mysql
-- 采购订单主表(分区表)
-- DROP TABLE tutorial.bl_po;
CREATE TABLE IF NOT EXISTS tutorial.bl_po (
 `id` Int64 COMMENT '采购订单主表-主键ID',
 `company_id` Int64 COMMENT '租赁公司ID(SAAS预留字段)',
 `bill_no` String COMMENT '单据编号',
 `status` String COMMENT '单据状态(10=制单 15=提交 20=审核 30=确认 99=取消 100=完结)',
 `creator` String COMMENT '创建人',
 `create_time` DateTime COMMENT '创建时间',
 `modifier` Nullable(String) COMMENT '修改人',
 `modify_time` Nullable(DateTime) COMMENT '修改时间',
 `auditor` Nullable(String) COMMENT '审核人',
 `audit_time` Nullable(DateTime) COMMENT '审核时间',
 `remarks` Nullable(String) COMMENT '备注',
 `update_time` DateTime COMMENT '记录更新时间'
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(create_time)
PRIMARY KEY bill_no
ORDER BY bill_no;

INSERT INTO tutorial.bl_po
(id, company_id, bill_no, status, creator, create_time, modifier, modify_time, auditor, audit_time, remarks, update_time)
VALUES
(1, 1, 'P20201102001', '99', '阳磊', '2020-11-02 12:12:12', Null, Null, '系统管理员', '2020-11-02 12:12:12', '', '2020-11-02 12:12:12');

INSERT INTO tutorial.bl_po
(id, company_id, bill_no, status, creator, create_time, modifier, modify_time, auditor, audit_time, remarks, update_time)
VALUES
(2, 1, 'P20201004001', '99', '阳磊', '2020-10-04 12:12:12', Null, Null, '系统管理员', '2020-10-04 12:12:12', '', '2020-10-04 12:12:12');

INSERT INTO tutorial.bl_po
(id, company_id, bill_no, status, creator, create_time, modifier, modify_time, auditor, audit_time, remarks, update_time)
VALUES
(3, 1, 'P20201103001', '99', '阳磊', '2020-11-03 12:12:12', Null, Null, '系统管理员', '2020-11-03 12:12:12', '', '2020-11-03 12:12:12');

-- 查询分区信息
SELECT database,table,partition_id,name,path
FROM system.parts p 
where database = 'tutorial' and table = 'bl_po';

-- 查询数据
SELECT * 
FROM tutorial.bl_po
where create_time > '2020-11-01 00:00:00';

-- 删除分区(会物理删除分区文件)
ALTER table tutorial.bl_po DROP PARTITION 202011;

-- 批量插入数据(只会产生一个物理文件目录)
INSERT INTO tutorial.bl_po
(id, company_id, bill_no, status, creator, create_time, modifier, modify_time, auditor, audit_time, remarks, update_time)
VALUES
(1, 1, 'P20201102001', '99', '阳磊', '2020-11-02 12:12:12', Null, Null, '系统管理员', '2020-11-02 12:12:12', '', '2020-11-02 12:12:12'),
(3, 1, 'P20201103001', '99', '阳磊', '2020-11-03 12:12:12', Null, Null, '系统管理员', '2020-11-03 12:12:12', '', '2020-11-03 12:12:12');

-- 复制分区表
CREATE table tutorial.bak_bl_po as tutorial.bl_po ENGINE = MergeTree() PARTITION BY toYYYYMM(create_time) ORDER BY bill_no;

-- 复制分区数据
alter table tutorial.bak_bl_po replace PARTITION 202011 from tutorial.bl_po;

-- 重置分区列数据
alter table tutorial.bak_bl_po clear column auditor in PARTITION 202011;

-- 卸载分区(文件未删除，被移动到 detached 文件夹)
alter table tutorial.bak_bl_po detach PARTITION 202011;

-- 装载分区
alter table tutorial.bak_bl_po attach PARTITION 202011;

-- 分区的备份与还原(后续学习...)
```

#### 副本表定义

```mysql
-- 采购订单主表(含副本的分区表-副本1)
-- DROP TABLE default.bl_po;
CREATE TABLE IF NOT EXISTS default.bl_po (
 `id` Int64 COMMENT '采购订单主表-主键ID',
 `company_id` Int64 COMMENT '租赁公司ID(SAAS预留字段)',
 `bill_no` String COMMENT '单据编号',
 `status` String COMMENT '单据状态(10=制单 15=提交 20=审核 30=确认 99=取消 100=完结)',
 `creator` String COMMENT '创建人',
 `create_time` DateTime COMMENT '创建时间',
 `modifier` Nullable(String) COMMENT '修改人',
 `modify_time` Nullable(DateTime) COMMENT '修改时间',
 `auditor` Nullable(String) COMMENT '审核人',
 `audit_time` Nullable(DateTime) COMMENT '审核时间',
 `remarks` Nullable(String) COMMENT '备注',
 `update_time` DateTime COMMENT '记录更新时间'
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/shard01/bl_po','cluster_chs01-01-1')
PARTITION BY toYYYYMM(create_time)
PRIMARY KEY bill_no
ORDER BY bill_no;

-- 采购订单主表(含副本的分区表-副本2)
-- DROP TABLE default.bl_po;
CREATE TABLE IF NOT EXISTS default.bl_po (
 `id` Int64 COMMENT '采购订单主表-主键ID',
 `company_id` Int64 COMMENT '租赁公司ID(SAAS预留字段)',
 `bill_no` String COMMENT '单据编号',
 `status` String COMMENT '单据状态(10=制单 15=提交 20=审核 30=确认 99=取消 100=完结)',
 `creator` String COMMENT '创建人',
 `create_time` DateTime COMMENT '创建时间',
 `modifier` Nullable(String) COMMENT '修改人',
 `modify_time` Nullable(DateTime) COMMENT '修改时间',
 `auditor` Nullable(String) COMMENT '审核人',
 `audit_time` Nullable(DateTime) COMMENT '审核时间',
 `remarks` Nullable(String) COMMENT '备注',
 `update_time` DateTime COMMENT '记录更新时间'
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/shard01/bl_po','cluster_chs01-01-2')
PARTITION BY toYYYYMM(create_time)
PRIMARY KEY bill_no
ORDER BY bill_no;

-- 查询 zookeeper 代理表(使用 SQL 的方式读取远端 zookeeper 内的数据)
SELECT * FROM system.zookeeper WHERE path = '/';
SELECT * FROM system.zookeeper WHERE path = '/clickhouse';
SELECT * FROM system.zookeeper WHERE path = '/clickhouse/tables/shard01';
SELECT * FROM system.zookeeper WHERE path = '/clickhouse/tables/shard01/bl_po';
SELECT * FROM system.zookeeper WHERE path = '/clickhouse/tables/shard01/bl_po/leader_election';

-- 在任意副本节点插入数据，发现其他的副本节点也会复制一份
```

#### 分片表定义

```mysql
-- 采购订单主表(集群表)
-- DROP TABLE IF EXISTS default.bl_po ON CLUSTER cluster_chs01;
CREATE TABLE IF NOT EXISTS default.bl_po ON CLUSTER cluster_chs01 (
 `id` Int64 COMMENT '采购订单主表-主键ID',
 `company_id` Int64 COMMENT '租赁公司ID(SAAS预留字段)',
 `bill_no` String COMMENT '单据编号',
 `status` String COMMENT '单据状态(10=制单 15=提交 20=审核 30=确认 99=取消 100=完结)',
 `creator` String COMMENT '创建人',
 `create_time` DateTime COMMENT '创建时间',
 `modifier` Nullable(String) COMMENT '修改人',
 `modify_time` Nullable(DateTime) COMMENT '修改时间',
 `auditor` Nullable(String) COMMENT '审核人',
 `audit_time` Nullable(DateTime) COMMENT '审核时间',
 `remarks` Nullable(String) COMMENT '备注',
 `update_time` DateTime COMMENT '记录更新时间'
) ENGINE = ReplicatedMergeTree('/clickhouse/tables/{shard}/bl_po','{replica}')
PARTITION BY toYYYYMM(create_time)
PRIMARY KEY bill_no
ORDER BY bill_no;

-- 采购订单主表(分片表)
-- DROP TABLE IF EXISTS default.bl_po_all ON CLUSTER cluster_chs01;
CREATE TABLE IF NOT EXISTS default.bl_po_all ON CLUSTER cluster_chs01 (
 `id` Int64 COMMENT '采购订单主表-主键ID',
 `company_id` Int64 COMMENT '租赁公司ID(SAAS预留字段)',
 `bill_no` String COMMENT '单据编号',
 `status` String COMMENT '单据状态(10=制单 15=提交 20=审核 30=确认 99=取消 100=完结)',
 `creator` String COMMENT '创建人',
 `create_time` DateTime COMMENT '创建时间',
 `modifier` Nullable(String) COMMENT '修改人',
 `modify_time` Nullable(DateTime) COMMENT '修改时间',
 `auditor` Nullable(String) COMMENT '审核人',
 `audit_time` Nullable(DateTime) COMMENT '审核时间',
 `remarks` Nullable(String) COMMENT '备注',
 `update_time` DateTime COMMENT '记录更新时间'
) ENGINE = Distributed(cluster_chs01, default, bl_po, rand());

-- 查询数据
SELECT * FROM default.bl_po_all;
```

### DML

```mysql
-- 插入数据
INSERT INTO `tutorial`.`bm_size`
(`id`, `company_id`, `size_no`, `size_code`, `size_name`, `owner_no`, `order_no`, `creator`, `create_time`, `modifier`, `modify_time`, `remarks`, `update_time`) 
VALUES 
(581, 1, '20220', '20220', '22.0cm', 'BL', 1, '系统管理员', '2019-03-12 12:12:12', NULL, NULL, NULL, '2019-04-25 10:24:27');
INSERT INTO `tutorial`.`bm_size`
(`id`, `company_id`, `size_no`, `size_code`, `size_name`, `owner_no`, `order_no`, `creator`, `create_time`, `modifier`, `modify_time`, `remarks`, `update_time`) 
VALUES 
(582, 1, '20225', '20225', '22.5cm', 'BL', 1, '系统管理员', '2019-03-12 12:12:12', NULL, NULL, NULL, '2019-04-25 10:24:27');

INSERT INTO `tutorial`.`bm_size`
(`id`, `company_id`, `size_no`, `size_code`, `size_name`, `owner_no`, `order_no`, `creator`, `create_time`,  `update_time`) 
FROM CSV \
'583', '1', '20230', '20230', '23.0cm', 'BL', '1', '系统管理员', '2019-03-12 12:12:12', '2019-04-25 10:24:27';

INSERT INTO `tutorial`.`bm_size`
(`id`, `company_id`, `size_no`, `size_code`, `size_name`, `owner_no`, `order_no`, `creator`, `create_time`,  `update_time`) 
FORMAT CSV 
'584', '1', '20235', '20235', '23.5cm', 'BL', '1', '系统管理员', '2019-03-12 12:12:12', '2019-04-25 10:24:27'
'585', '1', '20240', '20240', '24.0cm', 'BL', '1', '系统管理员', '2019-03-12 12:12:12', '2019-04-25 10:24:27';

insert into `tmp_bm_size` 
(
`id`, `company_id`, `size_no`, `size_code`, `size_name`, `owner_no`, `creator`, `create_time`, `modifier`, `modify_time`, `remarks`, `update_time`
)
select 
`id`, `company_id`, `size_no`, `size_code`, `size_name`, `owner_no`, `creator`, `create_time`, `modifier`, `modify_time`, `remarks`, `update_time`
from `tutorial`.`bm_size`;

-- 删除数据(不支持事务，实际是通过异步执行的)
alter table tutorial.bak_bl_po delete where id = 3;
-- 更新数据
alter table db_test.bm_size update size_name = '23.0cm' where id=582;
```

### DQL

```mysql
-- with子句(定义变量)
WITH 
	10 AS start,
	100 AS abc
SELECT number, number + abc
FROM system.numbers
WHERE number > start 
LIMIT 5;

-- 查看数据库占用空间大小
-- with子句(调用函数，重复的函数可以简化)
WITH SUM(c.data_uncompressed_bytes) AS bytes
SELECT c.database, formatReadableSize(bytes) AS disk_size
FROM system.columns c
GROUP BY c.database 
ORDER BY bytes DESC;

-- 查看数据库占用空间占比
-- with子句(定义子查询，注意：只能返回一行数据)
WITH 
(
	SELECT SUM(data_uncompressed_bytes) AS bytes FROM system.columns 
)
AS total_bytes,
SUM(c.data_uncompressed_bytes) AS bytes
SELECT c.database, round(bytes/total_bytes*100,2) AS disk_usage
FROM system.columns c
GROUP BY c.database 
ORDER BY disk_usage DESC;

-- 嵌套with
WITH round(disk_usage,2) AS disk_usage_rate
SELECT 
	database, disk_usage_rate
FROM 
(
	WITH 
	(
		SELECT SUM(data_uncompressed_bytes) AS bytes FROM system.columns 
	)
	AS total_bytes,
	SUM(c.data_uncompressed_bytes) AS bytes
	SELECT c.database, bytes/total_bytes*100 AS disk_usage
	FROM system.columns c
	GROUP BY c.database 
	ORDER BY disk_usage DESC
);

-- 生成序号
SELECT * FROM numbers(10);
SELECT * FROM numbers(100, 10);
SELECT * FROM system.numbers LIMIT 1,10;

-- 生成随机数(可用于造数据)
SELECT * 
FROM generateRandom('col_1 Array(Int8), col_2 String, col_3 Tuple(Decimal32(4)), col_4 DateTime', 1, 10, 3)
limit 20;
```

#### 执行计划

[尝鲜ClickHouse原生EXPLAIN查询功能](https://cloud.tencent.com/developer/article/1662230)

```sh
# 在clickhouse 20.6版本之前要查看SQL语句的执行计划需要设置日志级别为trace才能可以看到
clickhouse-client --send_logs_level=trace <<< 'SELECT COUNT(*) FROM tutorial.hits_v1' > /dev/null
clickhouse-client --send_logs_level=trace <<< 'SELECT * FROM tutorial.hits_v1' > /dev/null

# 在20.6版本引入了原生的执行计划的语法
# 执行计划的语法：：
# EXPLAIN [AST | SYNTAX | PLAN | PIPELINE] [setting = value, ...] SELECT ... [FORMAT ...]
# PLAN     用于查看执行计划，默认值。
#   header       打印计划中各个步骤的 head 说明，默认关闭，默认值0;
#   description  打印计划中各个步骤的描述，默认开启，默认值1；
#   actions      打印计划中各个步骤的详细信息，默认关闭，默认值0。
# AST      用于查看语法树;
# SYNTAX   用于优化语法;
# PIPELINE 用于查看 PIPELINE 计划。
#   header     打印计划中各个步骤的 head 说明，默认关闭;
#   graph     用DOT图形语言描述管道图，默认关闭，需要查看相关的图形需要配合graphviz查看；
#   actions   如果开启了graph，紧凑打印打，默认开启。
clickhouse-client -q "EXPLAIN PLAN SELECT COUNT(*) FROM tutorial.hits_v1"
clickhouse-client -q "EXPLAIN PIPELINE SELECT COUNT(*) FROM tutorial.hits_v1"
clickhouse-client -q "EXPLAIN header=1,actions=1,description=1 SELECT number from system.numbers limit 10;"
# 带参数的：需要结合graphviz图形工具查看。
clickhouse-client -q "EXPLAIN PIPELINE header=1,graph=1 SELECT sum(number) FROM numbers_mt(10000) GROUP BY number%5;"
```

### 函数

#### 算术函数

```mysql
SELECT toTypeName(0), toTypeName(0 + 0), toTypeName(0 + 0 + 0), toTypeName(0 + 0 + 0 + 0);

-- 加、减、乘、除、取模
select 
	100+200,200-100,200*100,200/11,200%11;
select 
	plus(200,100),minus(200,100),multiply(200,100),divide(200,11),modulo(200,11);
-- 整除向下取整
select 
	intDiv(200,11),intDivOrZero(200,11),intDivOrZero(200,0),moduloOrZero(200,0);
-- 取反、绝对值、最大公约数、最小公倍数
select 
	100,negate(-100),-100,negate(100),abs(-100),gcd(98,63),lcm(98,63);
-- 时间运算(在Date的情况下，和整数相加整数意味着添加相应的天数；对于DateTime，这意味着添加相应的秒数。)
SELECT 
	now(),now()+1,today(),today()+1;
-- 获取2个值的最小、最大值
SELECT 
	least(100, 200),greatest(100, 200);
-- 取整函数	
SELECT 
	floor(3.5),
	ceil(3.2),
	round(3.1415,2);
-- 随机函数
SELECT 
	rand(),rand64(),randConstant();
```

#### 类型转换行数

```mysql
SELECT 
	toInt8('32'),toInt16('255'),toInt32('65535'),toInt64('1000000000');

SELECT 
	toInt32('65535'),toInt32OrZero('65535a'),toInt32OrNull('65535a');

SELECT 
	toDecimal64('655.3535',4),toDecimal64OrZero('655.3535a',5),toDecimal64OrNull('655.3535a',5);

SELECT 
	toDate('2020-2-28'),toDateOrZero('2020-2-301'),toDateOrNull('2020-2-30d');

SELECT 
	now(),toString(now()),toString(now(),'Asia/Shanghai')

SELECT 
	'2016-06-15 23:00:00' AS timestamp,
	CAST(timestamp AS DateTime) AS datetime,
	CAST(timestamp AS Date) AS date,
	CAST(timestamp, 'String') AS string,
	CAST(timestamp, 'FixedString(22)') AS fixed_string;
```

#### 字符串函数

```mysql
SELECT 
	toFixedString('yang',8)||'lei' as myname,toStringCutToZero(myname)||'.' as cut;

-- 字符串判断
SELECT 
	empty(''),empty(' '),empty(NULL),
	notEmpty(''),notEmpty(' '),notEmpty(NULL),
	startsWith('abcde','ab'),
	endsWith('abcde','de');
-- 字符串长度
SELECT 
	length('abc啊啊啊'),lengthUTF8('abc啊啊啊'),
	char_length('abc啊啊啊'),character_length('abc啊啊啊');
-- 大小写转换
SELECT 
	lower('aBc'),lcase('aBc'),
	upper('aBc'),ucase('aBc'),
	reverse('abcde');
-- 字符串连接
SELECT 
	format('{0} {1}','hello','world'),
	concat('hello','world'),
	'helle' || 'world';
-- 字符串截取
SELECT 
	trimLeft('  abcde  '),
	trimRight('  abcde  '),
	trimBoth('  abcde  '),
	substring('abcde',3,3),
	appendTrailingCharIfAbsent('aabbcc','.');
-- 字符串分割于拼接
SELECT 
	splitByChar(',' , '123,456,789'),
	splitByString('||' , '123||456||789'),
	arrayStringConcat(splitByString('||' , '123||456||789'));
-- 字符串加解密
SELECT 
	base64Encode('https://clickhouse.tech/docs/v20.3/zh/query_language/functions/string_functions/') as encode,
	base64Decode(encode) decode,
	tryBase64Decode(NULL);	
-- 字符串搜索
SELECT 
	locate('aabbccddee','cc'),locate('aabbccddee','ff'),
	position('aabbccddee','cc'),position('aabbccddee','ff'),
	positionCaseInsensitive('aabbCCddee','cc'),positionCaseInsensitive('aabbCCddee','ff'),
	match('aabbccddeeaabbccddee','.+'),
	-- 正则表达式提取字符串
	extract('aabbccddeeaabbccddee','.+'),
	extractAll('aabbccddeeaabbccddee','.+');
-- 多字符串搜索
SELECT 
	multiSearchAllPositions('aabbccddeeaabbccddee',['cc','dd']),
	multiSearchFirstPosition('aabbccddeeaabbccddee',['cc','dd']),
	multiSearchFirstIndex('aabbccddeeaabbccddee',['cc','dd']),
	multiSearchAny('aabbccddeeaabbccddee',['cc','dd']),
	multiMatchAny('aabbccddeeaabbccddee',['.+']);
-- 复制字符串十次
SELECT 
	replaceRegexpOne('Hello, World!', '.*', '\\0\\0\\0\\0\\0\\0\\0\\0\\0\\0');
	
-- UUID函数
SELECT 
	generateUUIDv4(),toUUID('f4bf890f-f9dc-4332-ad5c-0c18e73f28e9');

-- 元素转换函数(适用于值转换为名称)
WITH 0 AS start
SELECT 
	number,
	transform(number, [2, 3], ['Yandex', 'Google'], 'Baidu') AS title
FROM system.numbers 
WHERE number > start
LIMIT 3;

-- IP函数
SELECT 
	IPv4StringToNum('192.168.244.1') AS num,toIPv4('192.168.244.1'),
	IPv4NumToString(num),IPv6NumToString(IPv4ToIPv6(num));

-- URL函数
SELECT 
	-- 返回域名并删除第一个'www.'
	domainWithoutWWW('https://clickhouse.tech/docs/zh/sql-reference/functions/url-functions/'),
	-- 返回顶级域名
	topLevelDomain('https://clickhouse.tech/docs/zh/sql-reference/functions/url-functions/'),
	cutToFirstSignificantSubdomain('https://clickhouse.tech/docs/zh/sql-reference/functions/url-functions/'),
	pathFull('https://clickhouse.tech/docs/zh/sql-reference/functions/url-functions/'),
	URLHierarchy('https://clickhouse.tech/docs/zh/sql-reference/functions/url-functions/'),
	-- 返回已经解码的URL
	decodeURLComponent('http://127.0.0.1:8123/?query=SELECT%201%3B'),
	-- 删除请求参数，问号也将被删除
	cutQueryString('http://127.0.0.1:8123/?query=SELECT%201%3B');

-- hash函数
SELECT 
	halfMD5('yanglei'),MD5('yanglei'),sipHash64('yanglei'),sipHash128('yanglei'),cityHash64('yanglei'),
	SHA1('yanglei'),SHA224('yanglei'),SHA256('yanglei');
SELECT intHash32(65535),intHash64(65535),URLHash('https://clickhouse.tech');

-- json函数(用到再研究)

-- 其他函数
SELECT 
	-- 当前查询的服务器的主机名
	hostName(),FQDN(),
	-- 当前查询的服务器的在线时间
	uptime(),
	-- 当前查询的服务器的版本
	version(),
	-- 当前查询的服务器的时区
	timezone(),
	formatReadableSize(filesystemAvailable()) AS "可用磁盘空间",
	formatReadableSize(filesystemCapacity()) AS "总磁盘空间",
	-- 当前数据库
	currentDatabase(),
	-- 当前用户
	currentUser(),
	-- 显示进度条
	bar(1000, 0, 10000, 20),
	-- 获取文件名
	basename('/data/long/path/to/abc.log'),
	-- 获取字符可见宽度
	visibleWidth('123嗷嗷'),
	-- 查询类型
	toTypeName('yanglei'),toColumnTypeName('yanglei'),dumpColumnStructure('yanglei'),
	-- 获取查询调用的块大小
	blockSize(),
	-- 在每个Block上休眠'seconds'秒，最大3秒
	sleep(3),
	-- 在每行上休眠'seconds'秒，最大3秒
	sleepEachRow(3);
```

#### 日期函数

```mysql
-- 时间增减间隔
SELECT 
	now() + toIntervalYear(1), now() + INTERVAL 1 Year,
	now() + toIntervalQuarter(1), now() + INTERVAL 1 Quarter,
	now() + toIntervalMonth(1), now() + INTERVAL 1 Month,
	now() + toIntervalWeek(1), now() + INTERVAL 1 Week,
	now() + toIntervalDay(1), now() + INTERVAL 1 Day,
	now() + toIntervalHour(1), now() + INTERVAL 1 Hour,
	now() + toIntervalMinute(1), now() + INTERVAL 1 Minute,
	now() + toIntervalSecond(1), now() + INTERVAL 1 Second;

-- 时间查询
SELECT 
	now(),today(),yesterday(),
	toYYYYMM(now()),
	toYYYYMMDD(now()),
	toYYYYMMDDhhmmss(now());

-- 日期格式化
SELECT 
	today(),replaceRegexpOne(toString(today()), '(\\d{4})-(\\d{2})-(\\d{2})', '\\2/\\3/\\1'),
	formatDateTime(now(),'%D');
	
-- 求时间间隔	
SELECT 
	dateDiff('day', yesterday(), today(), 'Asia/Shanghai')
```

#### 空值函数

```mysql
SELECT 
	toInt32OrNull('65535a') AS mynull,
	toInt32OrNull('65535b') AS mynull2,
	isNull(mynull),isNotNull(mynull),
	ifNull(mynull,100),
	coalesce(mynull,mynull2,100),
	CAST(mynull2, 'Nullable(Int32)'),
	NULLIF(2,2),NULLIF(2,3),NULLIF(2,mynull2),NULLIF(mynull,mynull2),
	assumeNotNull(mynull2),
	toTypeName(10),toTypeName(toNullable(10))
```

#### 字典函数

```mysql

```

#### 数组函数

```mysql
SELECT 
	length([1,2,3]),range(10),arrayConcat([1, 2], [3, 4], [5, 6]);
	
-- 数组行转列
SELECT 
	arrayJoin([1, 2, 3] AS src) AS dst, 'Hello ' || toString(dst);

-- 列转行(数组)
SELECT groupArray(y)
FROM 
(
	SELECT 1 x,100 y
	union all
	SELECT 2 x,200 y
	union all
	SELECT 2 x,NULL y
	union all
	SELECT 3 x,300 y
	union all
	SELECT 3 x,400 y
) a;
SELECT x,groupArray(y)
FROM 
(
	SELECT 1 x,100 y
	union all
	SELECT 2 x,200 y
	union all
	SELECT 2 x,NULL y
	union all
	SELECT 3 x,300 y
	union all
	SELECT 3 x,400 y
) a
group by x;
```

#### 经纬度函数(GEO函数)

```mysql
-- 返回地球表面的两点之间的距离，以米为单位。
SELECT greatCircleDistance(55.755831, 37.617673, -55.755831, -37.617673);

-- 将任何geohash编码的字符串解码为经度和纬度
SELECT geohashDecode('ezs42') AS res;
```

### 聚合函数

```mysql
SELECT 
	count(y),count(DISTINCT y),sum(y),max(y),min(y),avg(y),
	-- 统计唯一值的个数
	uniq(x),uniq(x,y)
	-- 选择最小y对应的x值
	argMin(x, y),
	-- 选择最大y对应的x值
	argMax(x, y),
	-- 选择第一个出现的值
	any(y),
	-- 选择一个出现最频繁的值
	anyHeavy(y),
	-- 选择最后一个出现的值
	anyLast(y),
	-- 计算累计求和数组
	groupArrayMovingSum(y),
	-- 计算累计求平均数组
	groupArrayMovingAvg(y),
	-- 返回topN数组
	topK(3)(y),topKWeighted(3)(y,y)
FROM 
(
	SELECT 1 x,200 y
	union all
	SELECT 2 x,100 y
	union all
	SELECT 2 x,NULL y
	union all
	SELECT 3 x,300 y
	union all
	SELECT 3 x,300 y
) a;

-- 按用户分组，根据每个用户的消费记录，按金额从大到小排序后分别设置行号
select 
    x,
    value,
    row_number
from
(
	SELECT 
		x,
		groupArray(y) value_list,
		arrayEnumerate(value_list) as index_list
	FROM 
	(
		SELECT *
		FROM 
		(
			SELECT 1 x,200 y
			union all
			SELECT 2 x,100 y
			union all
			SELECT 2 x,NULL y
			union all
			SELECT 3 x,300 y
			union all
			SELECT 3 x,100 y
		) a 
		order by y desc
	) a
	group by x
)
array join value_list as value,index_list as row_number
order by x;
```

### 字典

#### 外部扩展字典

需要注意的是字典的 `key` 必须是 `Uint64` 类型的，因我们大部分基础表的 `key` 都是 `String` 类型的，所以无法直接使用字典。

```mysql
DROP TABLE IF EXISTS tutorial.dic_table;
CREATE TABLE IF NOT EXISTS tutorial.dic_table (
  c_key UInt64 COMMENT '键',
  c_value String COMMENT '值',
 `update_time` DateTime COMMENT '记录更新时间'
) ENGINE = MergeTree()
PRIMARY KEY c_key
ORDER BY c_key;

insert into tutorial.dic_table values (1,'百丽',now());
insert into tutorial.dic_table values (2,'思加图',now());

CREATE DICTIONARY tutorial.dic_test
(
  c_key UInt64,
  c_value String
)
PRIMARY KEY c_key
-- 字典数据源
SOURCE(CLICKHOUSE(HOST 'localhost' PORT 9000 USER 'default' PASSWORD '' DB 'tutorial' TABLE 'dic_table'))
-- Memory layout configuration
LAYOUT(flat())
-- 更新频率(秒)
LIFETIME(MIN 10 MAX 30);

SELECT * 
FROM system.dictionaries;

SELECT * 
FROM tutorial.dic_test;

SELECT number,dictGet('tutorial.dic_test', 'c_value', number) val 
FROM numbers(5);

select * 
from numbers(5) as n 
inner join tutorial.dic_test on c_key=number;
```

#### 参考资料

[Clickhouse 数据字典](https://blog.csdn.net/vkingnew/article/details/106973674)

### 数据库管理

```mysql
-- 整理表
optimize table default.bl_po;

-- 查看集群信息
SELECT * FROM system.clusters;

-- 查看宏定义
SELECT * FROM system.macros;

-- 查看 clickhouse 的性能指标
SELECT * FROM system.metrics;
SELECT * FROM system.events;
SELECT * FROM system.asynchronous_metrics;

-- 查看用户的查询日志(默认关闭，需要开启)
SELECT * FROM system.query_log;

-- 数据导出文件备份(适合数据体量较小的表)
clickhouse-client --user=user_readonly --password=user_readonly --query="select * from bl_po_all" > bl_po_all.tsv
-- 备份数据导入
cat bl_po_all.tsv | clickhouse-client --user=user_readonly --password=user_readonly --query="insert into bl_po_all FORMAT TSV" --max_insert_block_size=100000

-- 冷备份(拷贝数据文件目录，适合大批量的数据备份)

-- 通过快照表备份(通过远程查询方式)
SELECT * FROM remote('172.17.209.53:61012','default','bl_po','user_readonly','user_readonly');

-- 查看数据库总容量、压缩率
select
    sum(rows) as "总行数",
    formatReadableSize(sum(data_uncompressed_bytes)) as "原始大小",
    formatReadableSize(sum(data_compressed_bytes)) as "压缩大小",
    round(sum(data_compressed_bytes) / sum(data_uncompressed_bytes) * 100, 0) "压缩率"
from system.parts;

-- 查看某个表的总容量、压缩率
select
	database as "库名",
    table as "表名",
    engine as "表引擎",
    sum(rows) as "总行数",
    formatReadableSize(sum(data_uncompressed_bytes)) as "原始大小",
    formatReadableSize(sum(data_compressed_bytes)) as "压缩大小",
    round(sum(data_compressed_bytes) / sum(data_uncompressed_bytes) * 100, 0) "压缩率"
from system.parts
where database in ('tutorial') and table in('hits_local')
group by database,table,engine;
```

#### RBAC权限分配

```mysql
-- 开启内省功能
-- 在计算机科学中，内省是指计算机程序在运行时（Run time）检查对象（Object）类型的一种能力，通常也可以称作运行时类型检查。
set allow_introspection_functions=1;

-- 查询有哪些权限项
select  * from system.privileges; 

-- 创建用户
CREATE USER admin IDENTIFIED WITH PLAINTEXT_PASSWORD BY 'Admin123' ON CLUSTER '{glayer}';
-- 赋予所有权限
GRANT ON CLUSTER '{glayer}' ALL ON *.* TO admin WITH GRANT OPTION;

-- 查询用户创建情况
show create user admin;
select name,id,storage,auth_type,host_ip from system.users where name in ('admin','default');
-- 查询用户赋权情况
show grants for admin;
select * from system.grants where user_name in ('admin','default');

-- 创建普通用户
CREATE USER user_test IDENTIFIED WITH PLAINTEXT_PASSWORD BY 'user_test' ON CLUSTER '{glayer}';
GRANT ON CLUSTER '{glayer}' SELECT,INSERT,ALTER UPDATE,ALTER DELETE ON db_test.* TO user_test;

-- 创建角色
CREATE ROLE role_app ON CLUSTER '{glayer}';
GRANT ON CLUSTER '{glayer}' SELECT,INSERT,ALTER UPDATE,ALTER DELETE ON ldp_lmp_ods.* TO role_app;
GRANT ON CLUSTER '{glayer}' SELECT,INSERT,ALTER UPDATE,ALTER DELETE ON ldp_lmd_ods.* TO role_app;

CREATE USER user_biz IDENTIFIED WITH PLAINTEXT_PASSWORD BY 'user_biz' ON CLUSTER '{glayer}';
GRANT role_app TO user_biz ON CLUSTER '{glayer}';
ALTER USER user_biz DEFAULT ROLE role_app ON CLUSTER '{glayer}';
CREATE SETTINGS PROFILE profile_max_memory_usage SETTINGS max_memory_usage=60000000000 TO role_app ON CLUSTER '{glayer}';

select * from system.role;
select * from system.role_grants;

-- 收回权限
-- REVOKE ON CLUSTER '{glayer}' SELECT,INSERT,ALTER MOVE PARTITION ON db_test.* FROM user_airflow;
```

[Clickhouse RBAC](https://blog.csdn.net/vkingnew/article/details/107308942)

[初探ClickHouse的RBAC权限功能](https://cloud.tencent.com/developer/article/1630927)

[clickhouse 用户权限设置](https://www.jianshu.com/p/3e08a7150fb1?utm_campaign=hugo)

[ClickHouse学习系列之二【用户权限管理】](https://www.cnblogs.com/zhoujinyi/p/12613026.html)

#### 配置文件

[ClickHouse学习系列之三【配置文件说明】](https://www.cnblogs.com/zhoujinyi/p/12627780.html)

### 数据库监控

[苏宁基于 ClickHouse 的大数据全链路监控实践](https://www.infoq.cn/article/ZJ5dFiYeywiK0DpG527C)

[clickhouse+chproxy+promethues](https://blog.csdn.net/yyoc97/article/details/106075220)

### 负载均衡

[chproxy](https://github.com/Vertamedia/chproxy/blob/master/README-CN.md)

[clickhouse + chproxy 集群搭建](https://www.jianshu.com/p/9498fedcfee7)

### 数据库备份与还原

#### 创建测试数据

```sql
CREATE database if not exists `db_backup_test` on cluster '{layer}';

-- DROP TABLE IF EXISTS db_backup_test.bl_po ON CLUSTER '{layer}';
CREATE TABLE IF NOT EXISTS db_backup_test.bl_po ON CLUSTER '{layer}' (
 `id` Int64 COMMENT '采购订单主表-主键ID',
 `company_id` Int64 COMMENT '租赁公司ID(SAAS预留字段)',
 `bill_no` String COMMENT '单据编号',
 `status` String COMMENT '单据状态(10=制单 15=提交 20=审核 30=确认 99=取消 100=完结)',
 `creator` String COMMENT '创建人',
 `create_time` DateTime COMMENT '创建时间',
 `modifier` Nullable(String) COMMENT '修改人',
 `modify_time` Nullable(DateTime) COMMENT '修改时间',
 `auditor` Nullable(String) COMMENT '审核人',
 `audit_time` Nullable(DateTime) COMMENT '审核时间',
 `remarks` Nullable(String) COMMENT '备注',
 `update_time` DateTime COMMENT '记录更新时间'
) ENGINE = ReplicatedMergeTree('/clickhouse/db_backup_test/tables/{shard}/bl_po','{replica}')
PARTITION BY toYYYYMM(create_time)
PRIMARY KEY bill_no
ORDER BY bill_no;

INSERT INTO db_backup_test.bl_po
(id, company_id, bill_no, status, creator, create_time, modifier, modify_time, auditor, audit_time, remarks, update_time)
VALUES
(1, 1, 'P20201102001', '99', '阳磊', '2020-11-02 12:12:12', Null, Null, '系统管理员', '2020-11-02 12:12:12', '', '2020-11-02 12:12:12');

INSERT INTO db_backup_test.bl_po
(id, company_id, bill_no, status, creator, create_time, modifier, modify_time, auditor, audit_time, remarks, update_time)
VALUES
(2, 1, 'P20201004001', '99', '阳磊', '2020-10-04 12:12:12', Null, Null, '系统管理员', '2020-10-04 12:12:12', '', '2020-10-04 12:12:12');

INSERT INTO db_backup_test.bl_po
(id, company_id, bill_no, status, creator, create_time, modifier, modify_time, auditor, audit_time, remarks, update_time)
VALUES
(3, 1, 'P20201103001', '99', '阳磊', '2020-11-03 12:12:12', Null, Null, '系统管理员', '2020-11-03 12:12:12', '', '2020-11-03 12:12:12');

SELECT * FROM db_backup_test.bl_po;
```

#### 导出csv方式备份

```mysql
-- 数据导出文件备份(适合数据体量较小的表)
-- 不能备份表结构，表结构需要有单独的备份
-- 备份文件比较大，需要进行压缩
clickhouse-client --host 172.17.209.53 --port 61011 --user=default --password=go2House --query="select * from db_backup_test.bl_po" > /var/lib/clickhouse/backup/172.17.209.53-61011-db_backup_test.bl_po.tsv
clickhouse-client --host 172.17.209.53 --port 61012 --user=default --password=go2House --query="select * from db_backup_test.bl_po" > /var/lib/clickhouse/backup/172.17.209.53-61012-db_backup_test.bl_po.tsv
clickhouse-client --host 172.17.209.53 --port 61021 --user=default --password=go2House --query="select * from db_backup_test.bl_po" > /var/lib/clickhouse/backup/172.17.209.53-61021-db_backup_test.bl_po.tsv
clickhouse-client --host 172.17.209.53 --port 61022 --user=default --password=go2House --query="select * from db_backup_test.bl_po" > /var/lib/clickhouse/backup/172.17.209.53-61022-db_backup_test.bl_po.tsv
-- 备份数据导入
-- 多个文件可以用 172.17.209.53-61011-db_backup_test*
cat /var/lib/clickhouse/backup/172.17.209.53-61011-db_backup_test.bl_po.tsv | clickhouse-client --host 172.17.209.53 --port 61011 --user=default --password=go2House --query="insert into db_backup_test.bl_po FORMAT TSV" --max_insert_block_size=100000
-- 多次导入数据会重复
-- 去重指令（需要ReplicatedReplacingMergeTree引擎表才能去重）
OPTIMIZE table db_backup_test.bl_po FINAL;
```

#### 表快照

```mysql
-- 适合临时备份单个表

-- drop table if exists db_backup_test.bl_po_local;
CREATE table db_backup_test.bl_po_local as db_backup_test.bl_po
ENGINE = ReplicatedMergeTree('/clickhouse/db_backup_test/tables/{shard}/bl_po_local','{replica}')
PARTITION BY toYYYYMM(create_time)
PRIMARY KEY bill_no
ORDER BY bill_no;

-- 本地表快照
insert into table db_backup_test.bl_po_local select * from db_backup_test.bl_po;

-- 远程表快照
insert into table db_backup_test.bl_po_local 
SELECT * FROM remote('172.17.209.53:61011','db_backup_test','bl_po','default','go2House');
```

#### 文件系统快照

某些本地文件系统提供快照功能（例如, [ZFS](https://en.wikipedia.org/wiki/ZFS)），但它们可能不是提供实时查询的最佳选择。 一个可能的解决方案是使用这种文件系统创建额外的副本，并将它们从 [分布](https://clickhouse.tech/docs/zh/engines/table-engines/special/distributed/) 用于以下目的的表 `SELECT` 查询。 任何修改数据的查询都无法访问此类副本上的快照。

暂不研究。

#### 冻结分区操作

ClickHouse允许使用 `ALTER TABLE ... FREEZE PARTITION ...` 查询以创建表分区的本地副本。 这是利用硬链接(hardlink)到 `/var/lib/clickhouse/shadow/` 文件夹中实现的，所以它通常不会占用旧数据的额外磁盘空间。  创建的文件副本不由ClickHouse服务器处理，所以你可以把它们留在那里：你将有一个简单的备份，不需要任何额外的外部系统，但它仍然会容易出现硬件问题。 出于这个原因，最好将它们远程复制到另一个位置，然后删除本地副本。  分布式文件系统和对象存储仍然是一个不错的选择，但是具有足够大容量的正常附加文件服务器也可以工作（在这种情况下，传输将通过网络文件系统 [rsync](https://en.wikipedia.org/wiki/Rsync)).

```sh
# 语法: ALTER TABLE table_name FREEZE [PARTITION partition_expr]
# 该操作为指定分区创建一个本地备份。
# 如果 PARTITION 语句省略，该操作会一次性为所有分区创建备份。整个备份过程不需要停止服务
# 注意：FREEZE PARTITION 只复制数据, 不备份元数据. 元数据默认在文件 /var/lib/clickhouse/metadata/database/table.sql
# 此方式个人认为也只适用于临时备份单个表

cat /var/lib/clickhouse/metadata/db_backup_test/bl_po_local.sql 

# 冻结表
alter table db_backup_test.bl_po_local freeze;
echo -n 'alter table db_backup_test.bl_po_local freeze;' | clickhouse-client --user=default --password=go2House

# 保存备份
mkdir -p /var/lib/clickhouse/backup/20210225
mv /var/lib/clickhouse/shadow/* /var/lib/clickhouse/backup/20210225
# 同时备份下表结构
cp /var/lib/clickhouse/metadata/db_backup_test/bl_po_local.sql /var/lib/clickhouse/backup/20210225

# 手动恢复
# 创建表，表不存在则根据备份的表结构创建，只需要将ATTACH修改为CREATE即可
cp -rl /var/lib/clickhouse/backup/20210225/2/data/db_backup_test/bl_po_local/* /var/lib/clickhouse/data/db_backup_test/bl_po_local/detached
chown -R clickhouse.clickhouse /var/lib/clickhouse/data/db_backup_test/bl_po_local/detached/*
echo 'alter table db_backup_test.bl_po_local attach partition 202010' | clickhouse-client --user=default --password=go2House
echo 'alter table db_backup_test.bl_po_local attach partition *' | clickhouse-client --user=default --password=go2House
```

ClickHouse应用文件系统硬链接来实现即时备份，而不会导致ClickHouse服务停机（或锁定）。这些硬链接能够进一步用于无效的备份存储。在反对硬链接的文件系统（例如本地文件系统或NFS）上，将cp与-l标记一起应用（或将rsync与–hard-links和–numeric-ids标记一起应用）以防止复制数据。 

当应用硬链接时，磁盘上的存储效率更高。因为它们依赖于硬链接，所以即便防止了重复使用磁盘空间，每个备份实际上都是“残缺”备份。

#### clickhouse-copier

`clickhouse-copier`是官方的数据迁移工具，用于多个集群之间的数据迁移。

[clickhouse-copier](https://clickhouse.tech/docs/zh/operations/utilities/clickhouse-copier/)

[clickhouse-copier 使用](https://hughsite.com/post/clickhouse-copier-usage.html)

[clickhouse-copier是如何进行数据迁移的](https://blog.csdn.net/weixin_39992480/article/details/111564982)

zookeeper.xml

```xml
<yandex>
    <logger>
        <level>trace</level>
        <size>100M</size>
        <count>3</count>
    </logger>

    <zookeeper>
        <node index="1">
            <host>zoo1</host>
            <port>2181</port>
        </node>
        <node index="2">
            <host>zoo2</host>
            <port>2181</port>
        </node>
        <node index="3">
            <host>zoo3</host>
            <port>2181</port>
        </node>
    </zookeeper>
</yandex>
```

task.xml

```xml
<yandex>
    <remote_servers>
        <src_cluster>
            <shard>
                <internal_replication>true</internal_replication>
                <replica>
                    <host>chs01011</host>
                    <port>9000</port>
                    <user>default</user>
                    <password>go2House</password>
                </replica>
                <replica>
                    <host>chs01012</host>
                    <port>9000</port>
                    <user>default</user>
                    <password>go2House</password>
                </replica>
            </shard>
        </src_cluster>
        <dst_cluster>
            <shard>
                <internal_replication>true</internal_replication>
                <replica>
                    <host>chs01021</host>
                    <port>9000</port>
                    <user>default</user>
                    <password>go2House</password>
                </replica>
                <replica>
                    <host>chs01022</host>
                    <port>9000</port>
                    <user>default</user>
                    <password>go2House</password>
                </replica>
            </shard>
        </dst_cluster>
    </remote_servers>
    <!-- 最大worker数 -->
    <max_workers>4</max_workers>
    <!-- fetch (pull) data 设置为只读 -->
    <settings_pull>
        <readonly>1</readonly>
    </settings_pull>
    <!-- insert (push) data 设置为读写 -->
    <settings_push>
        <readonly>0</readonly>
    </settings_push>

    <settings>
        <connect_timeout>3</connect_timeout>
        <insert_distributed_sync>1</insert_distributed_sync>
    </settings>
    
    <!-- 任务描述，可以配置多个，多个任务会顺序执行 -->
    <tables>
        <!-- 复制任务，名字自定义 -->
        <table_test_copy>
            <!-- cluster_pull要在remote_servers中有配置 -->
            <cluster_pull>src_cluster</cluster_pull>
            <database_pull>db_backup_test</database_pull>
            <table_pull>bl_po</table_pull>

            <!-- cluster_push也要在remote_servers中有配置 -->
            <cluster_push>dst_cluster</cluster_push>
            <database_push>db_backup_test</database_push>
            <table_push>bl_po</table_push>

            <!-- 目标集群没有表的情况下，会根据下面的配置来创建表 -->
            <engine>
            ENGINE = ReplicatedMergeTree('/clickhouse/db_backup_test/tables/{shard}/bl_po','{replica}')
            PARTITION BY toYYYYMM(create_time)
            PRIMARY KEY bill_no
            ORDER BY bill_no
            </engine>
            
            <!-- sharding_key用来控制用什么方式写入目的端集群 -->
            <sharding_key>rand()</sharding_key>
          
          	<!-- Optional expression that filter data while pull them from source servers -->
            <!-- <where_condition></where_condition> -->
          
          	<!-- <enabled_partitions> -->
            <!-- </enabled_partitions> -->
        </table_test_copy>
    </tables>
</yandex>
```

这个配置文件是用于开始前写入zk节点上的，如下：

```sh
bin/zkCli.sh create /clickhouse/copytasks ""
bin/zkCli.sh create /clickhouse/copytasks/test ""
bin/zkCli.sh create /clickhouse/copytasks/test/description "`cat task.xml`"

# 查看任务信息
bin/zkCli.sh get /clickhouse/copytasks/test/description

# 更新任务信息
bin/zkCli.sh set /clickhouse/copytasks/test/description "`cat task.xml`"

# 启动任务前台
clickhouse-copier --config /var/lib/clickhouse/backup/zookeeper.xml  --task-path /clickhouse/copytasks/test --base-dir /tmp/copylogs
# 启动任务后台
clickhouse-copier --daemon --config /var/lib/clickhouse/backup/zookeeper.xml  --task-path /clickhouse/copytasks/test --base-dir /tmp/copylogs
```

> 注意事项：
>
> - 原始表建表语句需要指定 `partition by`，在测试的时候由于没有指定，导致任务失败；
> - 对于物化视图复制，`table_pull` 表名需要添加 `.inner.` 前缀，不然任务会因找不到分区信息而失败，另外目标集群的物化视图需要提前创建好，不然由任务生成的表为普通表；
> - 复制过程使用的是 `insert into` 方式来批量写入数据，所以会触发物化视图的数据更新（如果数据量大的话，会很慢）；
> - 在集群间复制（多分片），数据会打乱；
> - 复制任务会在 zookeeper 中记录状态，但是表中已经处理过的 partition 下次任务不会再处理（即使有新数据）。
> - 如果做备份需要有1个ck备份集群，同步任务xml文件需要弄一个批量生成的工具。

#### clickhouse-backup

[AlexAkulov/clickhouse-backup](https://github.com/AlexAkulov/clickhouse-backup)

**特征**

- 轻松创建和还原所有或特定表的备份
- 在文件系统上高效存储多个备份
- 通过流压缩上传和下载
- 支持远程存储上的增量备份
- 适用于AWS，Azure，GCS，腾讯COS，FTP

**局限性**

- 支持高于1.1.54390和20.10之前的ClickHouse
- 仅MergeTree系列表引擎
- 备份“分层存储”或storage_policy不支持！
- 不支持在ClickHouse 20.10中默认启用的“原子”数据库引擎！
- 云存储上的最大备份大小为5TB
- AWS S3上的最大零件数为10,000（如果您的数据库大于1TB，则增加part_size）



docker部署：

```sh
# 查看要备份的表
docker run -it --rm --network host \
-v /data/docker_volumn/clickhouse/chs01012:/var/lib/clickhouse \
-v /data/clickhouse/backup_config.yml:/etc/clickhouse-backup/config.yml \
alexakulov/clickhouse-backup:0.6.4 tables

# 创建一个全备
docker run -it --rm --network host \
-v /data/docker_volumn/clickhouse/chs01012:/var/lib/clickhouse \
-v /data/clickhouse/backup_config.yml:/etc/clickhouse-backup/config.yml \
alexakulov/clickhouse-backup:0.6.4 create `date "+%Y-%m-%dT%H-%M-%S"`

# 单表备份

# 多表备份

# 查看备份
docker run -it --rm --network host \
-v /data/docker_volumn/clickhouse/chs01012:/var/lib/clickhouse \
-v /data/clickhouse/backup_config.yml:/etc/clickhouse-backup/config.yml \
alexakulov/clickhouse-backup:0.6.4 list

# 恢复(恢复被删除的表)
docker run -it --rm --network host \
-v /data/docker_volumn/clickhouse/chs01012:/var/lib/clickhouse \
-v /data/clickhouse/backup_config.yml:/etc/clickhouse-backup/config.yml \
alexakulov/clickhouse-backup:0.6.4 restore -t db_backup_test.bl_po 2021-03-04T22-23-40

# 恢复(只恢复表结构)

# 恢复(只恢复数据)
docker run -it --rm --network host \
-v /data/docker_volumn/clickhouse/chs01012:/var/lib/clickhouse \
-v /data/clickhouse/backup_config.yml:/etc/clickhouse-backup/config.yml \
alexakulov/clickhouse-backup:0.6.4 restore -t db_backup_test.bl_po --data 2021-03-04T22-23-40

# 恢复(在恢复前删除表)

# 二进制操作
clickhouse-backup tables
clickhouse-backup create $(date "+%Y-%m-%dT%H-%M-%S")
clickhouse-backup create -t db_test.hits_v1 $(date "+%Y-%m-%dT%H-%M-%S")
```

### 版本升级

[Clickhouse升级操作文档](http://www.clickhouse.com.cn/topic/6046d27cb06e5e0f21ba7a2c)

### 注意事项与要求

- 表名、字段名全部使用小写（ClickHouse的语法是大小写敏感的）

- 避免使用 select *

- with中使用子查询只能返回一行数据


### 问题记录

[ClickHouse删除数据之后再插入数据成功无报错但是查询不到数据](https://blog.csdn.net/qq_40341628/article/details/109388455?utm_medium=distribute.pc_relevant.none-task-blog-title-3&spm=1001.2101.3001.4242)

[Dberver 连接 ClickHouse:Cannot modify max_result_rows setting in readonly mode](https://blog.csdn.net/qq_18769269/article/details/106917575)

[ClickHouse连接超时的解决方法](https://blog.csdn.net/ws271/article/details/111914727)

### 参考资料

[clickhouse/yyoc97的专栏](https://blog.csdn.net/yyoc97/category_9053674.html)

[clickhouse/DBArtist的专栏](https://www.cnblogs.com/DBArtist/category/1835494.html)

[zk集群和clickhouse集群搭建](https://blog.csdn.net/qq_25864747/article/details/91997169)

[Clickhouse集群扩容收缩](https://aop.pub/artical/database/clickhouse/cluster-scale/)

[Clickhouse 在腾讯的应用实践](https://www.jiqizhixin.com/articles/2019-10-25-3)

[腾讯云 ClickHouse 如何实现自动化的数据均衡？](https://www.infoq.cn/article/mFqxEHjZUejlOwFpQCS9?utm_source=rss&utm_medium=article)

[用Docker快速上手Clickhouse](http://sineyuan.github.io/post/clickhouse-docker-quick-start/)

[ClickHouse - 多卷存储扩大存储容量(生产环境必备)](https://blog.csdn.net/jiangshouzhuang/article/details/103650360)

[DataX的Clickhouse读写插件](http://lab.orchina.org/design/clickhouse-reader-writer/)

[Clickhouse集群应用、分片、复制](https://www.jianshu.com/p/20639fdfdc99)

[ClickHouse分布式IN & JOIN 查询的避坑指南](https://www.it610.com/article/1278613838761050112.htm)

[clickhouse分析：zookeeper减压概述](https://blog.csdn.net/iceyung/article/details/107500073)

[ClickHouse多盘存储配置](https://cloud.tencent.com/developer/article/1649377)

[clickhouse 推荐配置](https://blog.csdn.net/qq_17202587/article/details/103697179)

[Clickhouse的实践之路](https://mp.weixin.qq.com/s?__biz=MzA5MTc0NTMwNQ==&mid=2650726172&idx=2&sn=5a05e571f9022ae6a61541ca1c251e35&chksm=887dc26abf0a4b7c7b66b942d2e77eb43b5ef0a1f8699b7ed1675ebe0b25abf1df9850daa726&scene=126&sessionid=1608169178&key=95883e0ca92484b1b6460365842f1e04febaf405a97264fd7cb0be5021dcdf345840bbcec8ca57636177a375add6aee6124b36cf3f22509b3e1a3e8f2ed07b6b3ee3784a26835f1ae85fd2753d8739dfe489db7625605beeb7a177aa8573ba238ff7ca7e20a7fb6bcd6babe445491f1d9d4ca43a2e9d61f0ebaec74bbaad714c&ascene=1&uin=MjEzOTUzMTk3Mg%3D%3D&devicetype=Windows+10+x64&version=63000039&lang=zh_CN&exportkey=A5D592Lrg7U8xu%2BEWNYH3Gc%3D&pass_ticket=UxG3GpbqaKnjZk8mb%2BTCcIQ8ZJ6sImc3H%2BDtMeNDaGuWJ55BjF2YMvpSapDGe9Ms&wx_header=0)

[云数据库ClickHouse二级索引-最佳实践](https://developer.aliyun.com/article/780402)

[配置ClickHouse分布式DDL记录自动清理](https://blog.csdn.net/nazeniwaresakini/article/details/107742717)

[如何处理ClickHouse超时问题](https://help.aliyun.com/document_detail/197622.html)

[关于clickhouse:简易教程ClickHouse-的数据备份与恢复](https://lequ7.com/guan-yu-clickhouse-jian-yi-jiao-cheng-clickhouse-de-shu-ju-bei-fen-yu-hui-fu.html)

[ClickHouse 备份恢复](https://www.jianshu.com/p/3ce923d0f767)

[Flink读写Clickhouse插件介绍](https://www.aboutyun.com/forum.php?spm=a2c6h.12873639.0.0.2025233chAHKka&mod=viewthread&tid=29271)

[ClickHouse实现数据有限更新(Limit Update)](https://mp.weixin.qq.com/s/EW6wMXxz3_gHkBdboBpPCg)

[ClickHouse基于复制的故障恢复测试](https://www.jianshu.com/p/2035691068c6?utm_campaign=maleskine&utm_content=note&utm_medium=reader_share&utm_source=weibo)

