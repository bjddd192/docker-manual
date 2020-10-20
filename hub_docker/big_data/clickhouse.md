# clickhouse

[官网](https://clickhouse.tech/)

[官方文档](https://clickhouse.tech/docs/v20.3/zh/)

[Yandex Cloud](https://console.cloud.yandex.com/)

### 官方镜像

[yandex/clickhouse-server](https://hub.docker.com/r/yandex/clickhouse-server/)

[yandex/clickhouse-client](https://hub.docker.com/r/yandex/clickhouse-client)

### 安装部署

#### 离线包下载

[官方下载](https://repo.yandex.ru/clickhouse/rpm/)

```sh
wget https://repo.yandex.ru/clickhouse/rpm/lts/x86_64/clickhouse-client-20.3.19.4-2.noarch.rpm
wget https://repo.yandex.ru/clickhouse/rpm/lts/x86_64/clickhouse-common-static-20.3.19.4-2.x86_64.rpm
wget https://repo.yandex.ru/clickhouse/rpm/lts/x86_64/clickhouse-server-20.3.19.4-2.noarch.rpm
wget https://repo.yandex.ru/clickhouse/rpm/lts/x86_64/clickhouse-common-static-dbg-20.3.19.4-2.x86_64.rpm
```



#### CentOS

```sh
sudo yum install yum-utils
sudo rpm --import https://repo.clickhouse.tech/CLICKHOUSE-KEY.GPG
sudo yum-config-manager --add-repo https://repo.clickhouse.tech/rpm/clickhouse.repo
sudo yum install clickhouse-server clickhouse-client

sudo /etc/init.d/clickhouse-server start
clickhouse-client
```

#### Docker部署

```sh
docker pull yandex/clickhouse-server:20.3.17.173
docker pull yandex/clickhouse-client:20.3.17.173
  
docker stop clickhouse-server && docker rm clickhouse-server 

# 端口说明：
# tcp_port   9000
# http_port  8123
# mysql_port 9004
# interserver_http_port(同步端口) 9009
docker run -d --name clickhouse-server -p 9000:9000 -p 8123:8123 -p 9004:9004 -p 9009:9009 --restart=always \
  --ulimit nofile=262144:262144 \
  -v /data/docker_volumn/clickhouse-server:/var/lib/clickhouse \
  yandex/clickhouse-server:20.3.17.173

# 持久化
# 数据目录：/var/lib/clickhouse
# 配置目录：/etc/clickhouse-server
# 日志目录：/var/log/clickhouse-server
docker cp clickhouse-server:/etc/clickhouse-server/config.xml /data/docker_volumn/clickhouse-server/config.xml

# 验证：
curl http://localhost:8123

# 从本地客户端连接Server
docker run -it --rm --link clickhouse-server:clickhouse-server \
  yandex/clickhouse-client:20.3.17.173 --host clickhouse-server
:) SHOW DATABASES;

SHOW DATABASES

┌─name────┐
│ default │
│ system  │
└─────────┘
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
FROM system.clusters

┌─cluster───────┬─shard_num─┬─shard_weight─┬─replica_num─┬─host_name─┬─host_address──┬─port─┬─is_local─┬
│ cluster_chs01 │         1 │            1 │           1 │ chs01011  │ 192.168.150.5 │ 9000 │        0 │
│ cluster_chs01 │         1 │            1 │           2 │ chs01012  │ 192.168.150.7 │ 9000 │        0 │
│ cluster_chs01 │         2 │            1 │           1 │ chs01021  │ 192.168.150.6 │ 9000 │        0 │
│ cluster_chs01 │         2 │            1 │           2 │ chs01022  │ 192.168.150.8 │ 9000 │        1 │
└───────────────┴───────────┴──────────────┴─────────────┴───────────┴───────────────┴──────┴──────────┴
```

##### 参考资料

[jneo8 / clickhouse-setup](https://github.com/jneo8/clickhouse-setup)

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

#### 表引擎

|  引擎类型   | 特点                                                         |
| :---------: | ------------------------------------------------------------ |
|  MergeTree  | 适用于高负载任务的最通用和功能最强大的表引擎。这些引擎的共同特点是可以快速插入数据并进行后续的后台数据处理。 MergeTree系列引擎支持数据复制，分区和一些其他引擎不支持的其他功能。<br />存储的数据按主键排序。<br />允许使用分区。<br />支持数据副本。<br />支持数据采样。 |
|     Log     | 具有最小功能的[轻量级引擎](https://clickhouse.tech/docs/v20.3/zh/operations/table_engines/log_family/)。当您需要快速写入许多小表（最多约100万行）并在以后整体读取它们时，该类型的引擎是最有效的。 |
| Intergation | 用于与其他的数据存储与处理系统集成的引擎。                   |
|             |                                                              |



#### 数据库操作

```sql
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

### DDL

#### 数据库定义

```sql
-- 创建一个默认引擎数据库
CREATE DATABASE IF NOT EXISTS tutorial;

-- 创建一个mysql引擎数据库(相当于dblink)
CREATE DATABASE IF NOT EXISTS db_test ENGINE = MySQL('10.0.30.39:3306', 'test', 'admin', 'Admttooerif0';
```

#### 基础表定义

```sql
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

```sql
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

```sql
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

#### 分片表定义

### DML

```sql
INSERT INTO `tutorial`.`bm_size`(`id`, `company_id`, `size_no`, `size_code`, `size_name`, `owner_no`, `order_no`, `creator`, `create_time`, `modifier`, `modify_time`, `remarks`, `update_time`) VALUES (581, 1, '20220', '20220', '22.0cm', 'BL', 1, '系统管理员', '2019-03-12 12:12:12', NULL, NULL, NULL, '2019-04-25 10:24:27');
INSERT INTO `tutorial`.`bm_size`(`id`, `company_id`, `size_no`, `size_code`, `size_name`, `owner_no`, `order_no`, `creator`, `create_time`, `modifier`, `modify_time`, `remarks`, `update_time`) VALUES (582, 1, '20225', '20225', '22.5cm', 'BL', 1, '系统管理员', '2019-03-12 12:12:12', NULL, NULL, NULL, '2019-04-25 10:24:27');

insert into `tmp_bm_size` 
(
`id`, `company_id`, `size_no`, `size_code`, `size_name`, `owner_no`, `creator`, `create_time`, `modifier`, `modify_time`, `remarks`, `update_time`
)
select 
`id`, `company_id`, `size_no`, `size_code`, `size_name`, `owner_no`, `creator`, `create_time`, `modifier`, `modify_time`, `remarks`, `update_time`
from `tutorial`.`bm_size`;
```



### 数据库管理

### 注意事项与要求

表名、字段名全部使用小写（ClickHouse的语法是大小写敏感的）

避免使用 select *



### 参考资料

[clickhouse/yyoc97的专栏](https://blog.csdn.net/yyoc97/category_9053674.html)

[Clickhouse集群扩容收缩](https://aop.pub/artical/database/clickhouse/cluster-scale/)

[Clickhouse 在腾讯的应用实践](https://www.jiqizhixin.com/articles/2019-10-25-3)

[腾讯云 ClickHouse 如何实现自动化的数据均衡？](https://www.infoq.cn/article/mFqxEHjZUejlOwFpQCS9?utm_source=rss&utm_medium=article)

[用Docker快速上手Clickhouse](http://sineyuan.github.io/post/clickhouse-docker-quick-start/)

[ClickHouse - 多卷存储扩大存储容量(生产环境必备)](https://blog.csdn.net/jiangshouzhuang/article/details/103650360)

[DataX的Clickhouse读写插件](http://lab.orchina.org/design/clickhouse-reader-writer/)

