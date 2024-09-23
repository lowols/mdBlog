---
title: 微服务常用组件ES-上
date: 2024-05-12 20:42:20
tags: ES
categories: 后端
---

**微服务常用组件ES-上**

B 站对应视频：

黑马商城作为一个电商项目，商品的搜索肯定是访问频率最高的页面之一。目前搜索功能是基于数据库的模糊搜索来实现的，存在很多问题。

**首先**，查询效率较低。

由于数据库模糊查询不走索引，在数据量较大的时候，查询性能很差。黑马商城的商品表中仅仅有不到 9 万条数据，基于数据库查询时，搜索接口的表现如图：

![](微服务常用组件ES-上/LZ9bbYm6PokIrzxknsqcMV4qnfg.png)

改为基于搜索引擎后，查询表现如下：

![](微服务常用组件ES-上/Zle0b2b6goZF0lxqu00c19DWn6d.png)

需要注意的是，数据库模糊查询随着表数据量的增多，查询性能的下降会非常明显，而搜索引擎的性能则不会随着数据增多而下降太多。目前仅 10 万不到的数据量差距就如此明显，如果数据量达到百万、千万、甚至上亿级别，这个性能差距会非常夸张。

**其次**，功能单一

数据库的模糊搜索功能单一，匹配条件非常苛刻，必须恰好包含用户搜索的关键字。而在搜索引擎中，用户输入出现个别错字，或者用拼音搜索、同义词搜索都能正确匹配到数据。

综上，在面临海量数据的搜索，或者有一些复杂搜索需求的时候，推荐使用专门的搜索引擎来实现搜索功能。

目前全球的搜索引擎技术排名如下：

![](微服务常用组件ES-上/KepFbdFzOonosGxQ7yxcSgjznug.png)

排名第一的就是我们今天要学习的 elasticsearch.

elasticsearch 是一款非常强大的开源搜索引擎，支持的功能非常多，例如：

![](微服务常用组件ES-上/TtGhbsvFjoFTyYxotidcTRHDnUb.png)
**代码搜索**
![](微服务常用组件ES-上/DO6rbyIztom0SgxEBIpc0DGNnjf.png)
**商品搜索**

![](微服务常用组件ES-上/Ngjqb46jyoAmUqxLd6acFepznXf.png)
**解决方案搜索**
![](微服务常用组件ES-上/CEejbZYEVoiwy2xZuxBcMkuhnah.png)
**地图搜索**

通过今天的学习大家要达成下列学习目标：

- 理解倒排索引原理
- 会使用 IK 分词器
- 理解索引库 Mapping 映射的属性含义
- 能创建索引库及映射
- 能实现文档的 CRUD

## 1.初识 elasticsearch

Elasticsearch 的官方网站如下：

本章我们一起来初步了解一下 Elasticsearch 的基本原理和一些基础概念。

### 1.1.认识和安装

Elasticsearch 是由 elastic 公司开发的一套搜索引擎技术，它是 elastic 技术栈中的一部分。完整的技术栈包括：

- Elasticsearch：用于数据存储、计算和搜索
- Logstash/Beats：用于数据收集
- Kibana：用于数据可视化

整套技术栈被称为 ELK，经常用来做日志收集、系统监控和状态分析等等：

![](微服务常用组件ES-上/TPSkbktndoGn2cxEXiEcY63dnfi.png)

整套技术栈的核心就是用来**存储**、**搜索**、**计算**的 Elasticsearch，因此我们接下来学习的核心也是 Elasticsearch。

我们要安装的内容包含 2 部分：

- elasticsearch：存储、搜索和运算
- kibana：图形化展示

首先 Elasticsearch 不用多说，是提供核心的数据存储、搜索、分析功能的。

然后是 Kibana，Elasticsearch 对外提供的是 Restful 风格的 API，任何操作都可以通过发送 http 请求来完成。不过 http 请求的方式、路径、还有请求参数的格式都有严格的规范。这些规范我们肯定记不住，因此我们要借助于 Kibana 这个服务。

Kibana 是 elastic 公司提供的用于操作 Elasticsearch 的可视化控制台。它的功能非常强大，包括：

- 对 Elasticsearch 数据的搜索、展示
- 对 Elasticsearch 数据的统计、聚合，并形成图形化报表、图形
- 对 Elasticsearch 的集群状态监控
- 它还提供了一个开发控制台（DevTools），在其中对 Elasticsearch 的 Restful 的 API 接口提供了**语法提示**

#### 1.1.1.安装 elasticsearch

通过下面的 Docker 命令即可安装单机版本的 elasticsearch：

```bash
docker run -d \
  --name es \
  -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
  -e "discovery.type=single-node" \
  -v es-data:/usr/share/elasticsearch/data \
  -v es-plugins:/usr/share/elasticsearch/plugins \
  --privileged \
  --network hm-net \
  -p 9200:9200 \
  -p 9300:9300 \
  elasticsearch:7.12.1
```

注意，这里我们采用的是 elasticsearch 的 7.12.1 版本，由于 8 以上版本的 JavaAPI 变化很大，在企业中应用并不广泛，企业中应用较多的还是 8 以下的版本。

如果拉取镜像困难，可以直接导入课前资料提供的镜像 tar 包：

![](微服务常用组件ES-上/AqOibER1hoV4RVxOLlNcKjp4nNe.png)

安装完成后，访问 9200 端口，即可看到响应的 Elasticsearch 服务的基本信息：

![](微服务常用组件ES-上/IWXfbJM8koM95mx6vAXcPD6AnGd.png)

#### 1.1.2.安装 Kibana

通过下面的 Docker 命令，即可部署 Kibana：

```bash
docker run -d \
--name kibana \
-e ELASTICSEARCH_HOSTS=http://es:9200 \
--network=hm-net \
-p 5601:5601  \
kibana:7.12.1
```

如果拉取镜像困难，可以直接导入课前资料提供的镜像 tar 包：

![](微服务常用组件ES-上/UQQqbKhYNocp0uxuaqvcblOvndc.png)

安装完成后，直接访问 5601 端口，即可看到控制台页面：

![](微服务常用组件ES-上/VeiUbgztYou2ZFxdnqGcXw8Znpd.png)

选择 `Explore on my own` 之后，进入主页面：

![](微服务常用组件ES-上/Z5zIbLU4koplP3xF1WZc7oSenQe.png)

然后选中 `Dev tools`，进入开发工具页面：

![](微服务常用组件ES-上/AXUPbdC1SocNanxDChlc4vh4nUb.png)

### 1.2.倒排索引

elasticsearch 之所以有如此高性能的搜索表现，正是得益于底层的倒排索引技术。那么什么是倒排索引呢？

**倒排**索引的概念是基于 MySQL 这样的**正向**索引而言的。

#### 1.2.1.正向索引

我们先来回顾一下正向索引。

例如有一张名为 `tb_goods` 的表：

| **id** | **title**      | **price** |
| ------ | -------------- | --------- |
| 1      | 小米手机       | 3499      |
| 2      | 华为手机       | 4999      |
| 3      | 华为小米充电器 | 49        |
| 4      | 小米手环       | 49        |
| ...    | ...            | ...       |

其中的 `id` 字段已经创建了索引，由于索引底层采用了 B+ 树结构，因此我们根据 id 搜索的速度会非常快。但是其他字段例如 `title`，只在叶子节点上存在。

因此要根据 `title` 搜索的时候只能遍历树中的每一个叶子节点，判断 title 数据是否符合要求。

比如用户的 SQL 语句为：

```sql
select * from tb_goods where title like '%手机%';
```

那搜索的大概流程如图：

![](微服务常用组件ES-上/PYCDbe5SnoUGiUxLTKLc08BKnDb.jpeg)

说明：

- 1）检查到搜索条件为 `like '%手机%'`，需要找到 `title` 中包含 `手机` 的数据
- 2）逐条遍历每行数据（每个叶子节点），比如第 1 次拿到 `id` 为 1 的数据
- 3）判断数据中的 `title` 字段值是否符合条件
- 4）如果符合则放入结果集，不符合则丢弃
- 5）回到步骤 1

综上，根据 id 精确匹配时，可以走索引，查询效率较高。而当搜索条件为模糊匹配时，由于索引无法生效，导致从索引查询退化为全表扫描，效率很差。

因此，正向索引适合于根据索引字段的精确搜索，不适合基于部分词条的模糊匹配。

而倒排索引恰好解决的就是根据部分词条模糊匹配的问题。

#### 1.2.2.倒排索引

倒排索引中有两个非常重要的概念：

- 文档（`Document`）：用来搜索的数据，其中的每一条数据就是一个文档。例如一个网页、一个商品信息
- 词条（`Term`）：对文档数据或用户搜索数据，利用某种算法分词，得到的具备含义的词语就是词条。例如：我是中国人，就可以分为：我、是、中国人、中国、国人这样的几个词条

**创建倒排索引**是对正向索引的一种特殊处理和应用，流程如下：

- 将每一个文档的数据利用**分词算法**根据语义拆分，得到一个个词条
- 创建表，每行数据包括词条、词条所在文档 id、位置等信息
- 因为词条唯一性，可以给词条创建**正向**索引

此时形成的这张以词条为索引的表，就是倒排索引表，两者对比如下：

**正向索引**

| **id（索引）** | **title**      | **price** |
| -------------- | -------------- | --------- |
| 1              | 小米手机       | 3499      |
| 2              | 华为手机       | 4999      |
| 3              | 华为小米充电器 | 49        |
| 4              | 小米手环       | 49        |
| ...            | ...            | ...       |

**倒排索引**

| **词条（索引）** | **文档 id** |
| ---------------- | ----------- |
| 小米             | 1，3，4     |
| 手机             | 1，2        |
| 华为             | 2，3        |
| 充电器           | 3           |
| 手环             | 4           |

倒排索引的**搜索流程**如下（以搜索"华为手机"为例），如图：

![](微服务常用组件ES-上/QFbAbDvVGow9EWxDad9cosYEn8f.jpeg)

流程描述：

1）用户输入条件 `"华为手机"` 进行搜索。

2）对用户输入条件**分词**，得到词条：`华为`、`手机`。

3）拿着词条在倒排索引中查找（**由于词条有索引，查询效率很高**），即可得到包含词条的文档 id：`1、2、3`。

4）拿着文档 `id` 到正向索引中查找具体文档即可（由于 `id` 也有索引，查询效率也很高）。

虽然要先查询倒排索引，再查询倒排索引，但是无论是词条、还是文档 id 都建立了索引，查询速度非常快！无需全表扫描。

#### 1.2.3.正向和倒排

那么为什么一个叫做正向索引，一个叫做倒排索引呢？

- **正向索引**是最传统的，根据 id 索引的方式。但根据词条查询时，必须先逐条获取每个文档，然后判断文档中是否包含所需要的词条，是**根据文档找词条的过程**。
- 而**倒排索引**则相反，是先找到用户要搜索的词条，根据词条得到保护词条的文档的 id，然后根据 id 获取文档。是**根据词条找文档的过程**。

是不是恰好反过来了？

那么两者方式的优缺点是什么呢？

**正向索引**：

- 优点：

  - 可以给多个字段创建索引
  - 根据索引字段搜索、排序速度非常快
- 缺点：

  - 根据非索引字段，或者索引字段中的部分词条查找时，只能全表扫描。

**倒排索引**：

- 优点：

  - 根据词条搜索、模糊搜索时，速度非常快
- 缺点：

  - 只能给词条创建索引，而不是字段
  - 无法根据字段做排序

### 1.3.基础概念

elasticsearch 中有很多独有的概念，与 mysql 中略有差别，但也有相似之处。

#### 1.3.1.文档和字段

elasticsearch 是面向**文档（Document）**存储的，可以是数据库中的一条商品数据，一个订单信息。文档数据会被序列化为 `json` 格式后存储在 `elasticsearch` 中：

![](微服务常用组件ES-上/BurZbt08qoYxjfxZJLscIh21nah.png)

```json
{
    "id": 1,
    "title": "小米手机",
    "price": 3499
}
{
    "id": 2,
    "title": "华为手机",
    "price": 4999
}
{
    "id": 3,
    "title": "华为小米充电器",
    "price": 49
}
{
    "id": 4,
    "title": "小米手环",
    "price": 299
}
```

因此，原本数据库中的一行数据就是 ES 中的一个 JSON 文档；而数据库中每行数据都包含很多列，这些列就转换为 JSON 文档中的**字段（Field）**。

#### 1.3.2.索引和映射

随着业务发展，需要在 es 中存储的文档也会越来越多，比如有商品的文档、用户的文档、订单文档等等：

![](微服务常用组件ES-上/UaC7bTxFSoH4P9xZyddcXU0Xnqb.png)

所有文档都散乱存放显然非常混乱，也不方便管理。

因此，我们要将类型相同的文档集中在一起管理，称为**索引（Index）**。例如：

**商品索引**

```json
{
    "id": 1,
    "title": "小米手机",
    "price": 3499
}

{
    "id": 2,
    "title": "华为手机",
    "price": 4999
}

{
    "id": 3,
    "title": "三星手机",
    "price": 3999
}
```

**用户索引**

```json
{
    "id": 101,
    "name": "张三",
    "age": 21
}

{
    "id": 102,
    "name": "李四",
    "age": 24
}

{
    "id": 103,
    "name": "麻子",
    "age": 18
}
```

**订单索引**

```json
{
    "id": 10,
    "userId": 101,
    "goodsId": 1,
    "totalFee": 294
}

{
    "id": 11,
    "userId": 102,
    "goodsId": 2,
    "totalFee": 328
}
```

- 所有用户文档，就可以组织在一起，称为用户的索引；
- 所有商品的文档，可以组织在一起，称为商品的索引；
- 所有订单的文档，可以组织在一起，称为订单的索引；

因此，我们可以把索引当做是数据库中的表。

数据库的表会有约束信息，用来定义表的结构、字段的名称、类型等信息。因此，索引库中就有**映射（mapping）**，是索引中文档的字段约束信息，类似表的结构约束。

#### 1.3.3.mysql 与 elasticsearch

我们统一的把 mysql 与 elasticsearch 的概念做一下对比：

| **MySQL** | **Elasticsearch** | **说明**                                                     |
| --------- | ----------------- | ------------------------------------------------------------ |
| Table     | Index             | 索引(index)，就是文档的集合，类似数据库的表(table)           |
| Row       | Document          | 文档（Document），就是一条条的数据，类似数据库中的行（Row），文档都是 JSON 格式 |
| Column    | Field             | 字段（Field），就是 JSON 文档中的字段，类似数据库中的列（Column） |
| Schema    | Mapping           | Mapping（映射）是索引中文档的约束，例如字段类型约束。类似数据库的表结构（Schema） |
| SQL       | DSL               | DSL 是 elasticsearch 提供的 JSON 风格的请求语句，用来操作 elasticsearch，实现 CRUD |

如图：

![](微服务常用组件ES-上/WJksbzTtCok6xxxjJIBcEd46nph.png)

那是不是说，我们学习了 elasticsearch 就不再需要 mysql 了呢？

并不是如此，两者各自有自己的擅长之处：

- Mysql：擅长事务类型操作，可以确保数据的安全和一致性
- Elasticsearch：擅长海量数据的搜索、分析、计算

因此在企业中，往往是两者结合使用：

- 对安全性要求较高的写操作，使用 mysql 实现
- 对查询性能要求较高的搜索需求，使用 elasticsearch 实现
- 两者再基于某种方式，实现数据的同步，保证一致性

![](微服务常用组件ES-上/ZTnGb1dMtobB3PxyD55cYly2nFb.png)

### 1.4.IK 分词器

Elasticsearch 的关键就是倒排索引，而倒排索引依赖于对文档内容的分词，而分词则需要高效、精准的分词算法，IK 分词器就是这样一个中文分词算法。

#### 1.4.1.安装 IK 分词器

**方案一**：在线安装

运行一个命令即可：

```shell
docker exec -it es ./bin/elasticsearch-plugin  install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.12.1/elasticsearch-analysis-ik-7.12.1.zip
```

然后重启 es 容器：

```shell
docker restart es
```

**方案二**：离线安装

如果网速较差，也可以选择离线安装。

首先，查看之前安装的 Elasticsearch 容器的 plugins 数据卷目录：

```shell
docker volume inspect es-plugins
```

结果如下：

```json
[
    {
        "CreatedAt": "2024-11-06T10:06:34+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/es-plugins/_data",
        "Name": "es-plugins",
        "Options": null,
        "Scope": "local"
    }
]
```

可以看到 elasticsearch 的插件挂载到了 `/var/lib/docker/volumes/es-plugins/_data` 这个目录。我们需要把 IK 分词器上传至这个目录。

找到课前资料提供的 ik 分词器插件，课前资料提供了 `7.12.1` 版本的 ik 分词器压缩文件，你需要对其解压：

![](微服务常用组件ES-上/OzZGb3WvIoolabxUAQCcouPsnid.png)

然后上传至虚拟机的 `/var/lib/docker/volumes/es-plugins/_data` 这个目录：

![](微服务常用组件ES-上/A8fgbyMFzohtr1xTap4cc0kQnx8.png)

最后，重启 es 容器：

```shell
docker restart es
```

#### 1.4.2.使用 IK 分词器

IK 分词器包含两种模式：

- `ik_smart`：智能语义切分
- `ik_max_word`：最细粒度切分

我们在 Kibana 的 DevTools 上来测试分词器，首先测试 Elasticsearch 官方提供的标准分词器：

```json
POST /_analyze
{
  "analyzer": "standard",
  "text": "黑马程序员学习java太棒了"
}
```

结果如下：

```json
{
  "tokens" : [
    {
      "token" : "黑",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "<IDEOGRAPHIC>",
      "position" : 0
    },
    {
      "token" : "马",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "<IDEOGRAPHIC>",
      "position" : 1
    },
    {
      "token" : "程",
      "start_offset" : 2,
      "end_offset" : 3,
      "type" : "<IDEOGRAPHIC>",
      "position" : 2
    },
    {
      "token" : "序",
      "start_offset" : 3,
      "end_offset" : 4,
      "type" : "<IDEOGRAPHIC>",
      "position" : 3
    },
    {
      "token" : "员",
      "start_offset" : 4,
      "end_offset" : 5,
      "type" : "<IDEOGRAPHIC>",
      "position" : 4
    },
    {
      "token" : "学",
      "start_offset" : 5,
      "end_offset" : 6,
      "type" : "<IDEOGRAPHIC>",
      "position" : 5
    },
    {
      "token" : "习",
      "start_offset" : 6,
      "end_offset" : 7,
      "type" : "<IDEOGRAPHIC>",
      "position" : 6
    },
    {
      "token" : "java",
      "start_offset" : 7,
      "end_offset" : 11,
      "type" : "<ALPHANUM>",
      "position" : 7
    },
    {
      "token" : "太",
      "start_offset" : 11,
      "end_offset" : 12,
      "type" : "<IDEOGRAPHIC>",
      "position" : 8
    },
    {
      "token" : "棒",
      "start_offset" : 12,
      "end_offset" : 13,
      "type" : "<IDEOGRAPHIC>",
      "position" : 9
    },
    {
      "token" : "了",
      "start_offset" : 13,
      "end_offset" : 14,
      "type" : "<IDEOGRAPHIC>",
      "position" : 10
    }
  ]
}
```

可以看到，标准分词器智能 1 字 1 词条，无法正确对中文做分词。

我们再测试 IK 分词器：

```json
POST /_analyze
{
  "analyzer": "ik_smart",
  "text": "黑马程序员学习java太棒了"
}
```

执行结果如下：

```json
{
  "tokens" : [
    {
      "token" : "黑马",
      "start_offset" : 0,
      "end_offset" : 2,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "程序员",
      "start_offset" : 2,
      "end_offset" : 5,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "学习",
      "start_offset" : 5,
      "end_offset" : 7,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "java",
      "start_offset" : 7,
      "end_offset" : 11,
      "type" : "ENGLISH",
      "position" : 3
    },
    {
      "token" : "太棒了",
      "start_offset" : 11,
      "end_offset" : 14,
      "type" : "CN_WORD",
      "position" : 4
    }
  ]
}
```

#### 1.4.3.拓展词典

随着互联网的发展，“造词运动”也越发的频繁。出现了很多新的词语，在原有的词汇列表中并不存在。比如：“泰裤辣”，“传智播客” 等。

IK 分词器无法对这些词汇分词，测试一下：

```json
POST /_analyze
{
  "analyzer": "ik_max_word",
  "text": "传智播客开设大学,真的泰裤辣！"
}
```

结果：

```json
{
  "tokens" : [
    {
      "token" : "传",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "CN_CHAR",
      "position" : 0
    },
    {
      "token" : "智",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "播",
      "start_offset" : 2,
      "end_offset" : 3,
      "type" : "CN_CHAR",
      "position" : 2
    },
    {
      "token" : "客",
      "start_offset" : 3,
      "end_offset" : 4,
      "type" : "CN_CHAR",
      "position" : 3
    },
    {
      "token" : "开设",
      "start_offset" : 4,
      "end_offset" : 6,
      "type" : "CN_WORD",
      "position" : 4
    },
    {
      "token" : "大学",
      "start_offset" : 6,
      "end_offset" : 8,
      "type" : "CN_WORD",
      "position" : 5
    },
    {
      "token" : "真的",
      "start_offset" : 9,
      "end_offset" : 11,
      "type" : "CN_WORD",
      "position" : 6
    },
    {
      "token" : "泰",
      "start_offset" : 11,
      "end_offset" : 12,
      "type" : "CN_CHAR",
      "position" : 7
    },
    {
      "token" : "裤",
      "start_offset" : 12,
      "end_offset" : 13,
      "type" : "CN_CHAR",
      "position" : 8
    },
    {
      "token" : "辣",
      "start_offset" : 13,
      "end_offset" : 14,
      "type" : "CN_CHAR",
      "position" : 9
    }
  ]
}
```

可以看到，`传智播客` 和 `泰裤辣` 都无法正确分词。

所以要想正确分词，IK 分词器的词库也需要不断的更新，IK 分词器提供了扩展词汇的功能。

1）打开 IK 分词器 config 目录：

![](微服务常用组件ES-上/ReJubTDGjoIFVGxbVjlcQ8g2nyg.png)

注意，如果采用在线安装的通过，默认是没有 config 目录的，需要把课前资料提供的 ik 下的 config 上传至对应目录。

2）在 IKAnalyzer.cfg.xml 配置文件内容添加：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
<properties>
        <comment>IK Analyzer 扩展配置</comment>
        <!--用户可以在这里配置自己的扩展字典 *** 添加扩展词典-->
        <entry key="ext_dict">ext.dic</entry>
</properties>
```

3）在 IK 分词器的 config 目录新建一个 `ext.dic`，可以参考 config 目录下复制一个配置文件进行修改

```
传智播客
泰裤辣
```

4）重启 elasticsearch

```shell
docker restart es

# 查看 日志
docker logs -f elasticsearch
```

再次测试，可以发现 `传智播客` 和 `泰裤辣` 都正确分词了：

```json
{
  "tokens" : [
    {
      "token" : "传智播客",
      "start_offset" : 0,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 0
    },
    {
      "token" : "开设",
      "start_offset" : 4,
      "end_offset" : 6,
      "type" : "CN_WORD",
      "position" : 1
    },
    {
      "token" : "大学",
      "start_offset" : 6,
      "end_offset" : 8,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "真的",
      "start_offset" : 9,
      "end_offset" : 11,
      "type" : "CN_WORD",
      "position" : 3
    },
    {
      "token" : "泰裤辣",
      "start_offset" : 11,
      "end_offset" : 14,
      "type" : "CN_WORD",
      "position" : 4
    }
  ]
}
```

#### 1.4.4.总结

分词器的作用是什么？

- 创建倒排索引时，对文档分词
- 用户搜索时，对输入的内容分词

IK 分词器有几种模式？

- `ik_smart`：智能切分，粗粒度
- `ik_max_word`：最细切分，细粒度

IK 分词器如何拓展词条？如何停用词条？

- 利用 config 目录的 `IkAnalyzer.cfg.xml` 文件添加拓展词典和停用词典
- 在词典中添加拓展词条或者停用词条

## 2.索引库操作

Index 就类似数据库表，Mapping 映射就类似表的结构。我们要向 es 中存储数据，必须先创建 Index 和 Mapping

### 2.1.Mapping 映射属性

Mapping 是对索引库中文档的约束，常见的 Mapping 属性包括：

- `type`：字段数据类型，常见的简单类型有：

  - 字符串：`text`（可分词的文本）、`keyword`（精确值，例如：品牌、国家、ip 地址）
  - 数值：`long`、`integer`、`short`、`byte`、`double`、`float`、
  - 布尔：`boolean`
  - 日期：`date`
  - 对象：`object`
- `index`：是否创建索引，默认为 `true`
- `analyzer`：使用哪种分词器
- `properties`：该字段的子字段

例如下面的 json 文档：

```json
{
    "age": 21,
    "weight": 52.1,
    "isMarried": false,
    "info": "黑马程序员Java讲师",
    "email": "zy@itcast.cn",
    "score": [99.1, 99.5, 98.9],
    "name": {
        "firstName": "云",
        "lastName": "赵"
    }
}
```

对应的每个字段映射（Mapping）：

| **字段名** |           | **字段类型** | **类型说明**       | **是否****参与搜索** | **是否****参与分词** | **分词器** |
| ---------- | --------- | ------------ | ------------------ | -------------------- | -------------------- | ---------- |
| age        |           | `integer`    | 整数               | - [x]                | - [ ]                | ——         |
| weight     |           | `float`      | 浮点数             | - [x]                | - [ ]                | ——         |
| isMarried  |           | `boolean`    | 布尔               | - [x]                | - [ ]                | ——         |
| info       |           | `text`       | 字符串，但需要分词 | - [x]                | - [x]                | IK         |
| email      |           | `keyword`    | 字符串，但是不分词 | - [ ]                | - [ ]                | ——         |
| score      |           | `float`      | 只看数组中元素类型 | - [x]                | - [ ]                | ——         |
| name       | firstName | `keyword`    | 字符串，但是不分词 | - [x]                | - [ ]                | ——         |
|            | lastName  | `keyword`    | 字符串，但是不分词 | - [x]                | - [ ]                | ——         |

### 2.2.索引库的 CRUD

由于 Elasticsearch 采用的是 Restful 风格的 API，因此其请求方式和路径相对都比较规范，而且请求参数也都采用 JSON 风格。

我们直接基于 Kibana 的 DevTools 来编写请求做测试，由于有语法提示，会非常方便。

#### 2.2.1.创建索引库和映射

**基本语法**：

- 请求方式：`PUT`
- 请求路径：`/索引库名`，可以自定义
- 请求参数：`mapping` 映射

**格式**：

```json
PUT /索引库名称
{
  "mappings": {
    "properties": {
      "字段名":{
        "type": "text",
        "analyzer": "ik_smart"
      },
      "字段名2":{
        "type": "keyword",
        "index": "false"
      },
      "字段名3":{
        "properties": {
          "子字段": {
            "type": "keyword"
          }
        }
      },
      // ...略
    }
  }
}
```

**示例**：

```json
# PUT /heima
{
  "mappings": {
    "properties": {
      "info":{
        "type": "text",
        "analyzer": "ik_smart"
      },
      "email":{
        "type": "keyword",
        "index": "false"
      },
      "name":{
        "properties": {
          "firstName": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

#### 2.2.2.查询索引库

**基本语法**：

- 请求方式：GET
- 请求路径：/索引库名
- 请求参数：无

**格式**：

```
GET /索引库名
```

**示例**：

```
GET /heima
```

#### 2.2.3.修改索引库

倒排索引结构虽然不复杂，但是一旦数据结构改变（比如改变了分词器），就需要重新创建倒排索引，这简直是灾难。因此索引库**一旦创建，无法修改 mapping**。

虽然无法修改 mapping 中已有的字段，但是却允许添加新的字段到 mapping 中，因为不会对倒排索引产生影响。因此修改索引库能做的就是向索引库中添加新字段，或者更新索引库的基础属性。

**语法说明**：

```json
PUT /索引库名/_mapping
{
  "properties": {
    "新字段名":{
      "type": "integer"
    }
  }
}
```

**示例**：

```json
PUT /heima/_mapping
{
  "properties": {
    "age":{
      "type": "integer"
    }
  }
}
```

#### 2.2.4.删除索引库

**语法：**

- 请求方式：DELETE
- 请求路径：/索引库名
- 请求参数：无

**格式：**

```
DELETE /索引库名
```

示例：

```
DELETE /heima
```

#### 2.2.5.总结

索引库操作有哪些？

- 创建索引库：PUT /索引库名
- 查询索引库：GET /索引库名
- 删除索引库：DELETE /索引库名
- 修改索引库，添加字段：PUT /索引库名/_mapping

可以看到，对索引库的操作基本遵循的 Restful 的风格，因此 API 接口非常统一，方便记忆。

## 3.文档操作

有了索引库，接下来就可以向索引库中添加数据了。

Elasticsearch 中的数据其实就是 JSON 风格的文档。操作文档自然保护 `增`、`删`、`改`、`查` 等几种常见操作，我们分别来学习。

### 3.1.新增文档

**语法：**

```json
POST /索引库名/_doc/文档id
{
    "字段1": "值1",
    "字段2": "值2",
    "字段3": {
        "子属性1": "值3",
        "子属性2": "值4"
    },
}
```

**示例：**

```json
POST /heima/_doc/1
{
    "info": "黑马程序员Java讲师",
    "email": "zy@itcast.cn",
    "name": {
        "firstName": "云",
        "lastName": "赵"
    }
}
```

**响应：**

![](微服务常用组件ES-上/MuaRbZh0ooPlXdxQ9DdcEgOPn6d.png)

### 3.2.查询文档

根据 rest 风格，新增是 post，查询应该是 get，不过查询一般都需要条件，这里我们把文档 id 带上。

**语法：**

```json
GET /{索引库名称}/_doc/{id}
```

**示例：**

```javascript
GET /heima/_doc/1
```

**查看结果：**

![](微服务常用组件ES-上/MPjabItRKo14qaxGkLTcZyVznAd.png)

### 3.3.删除文档

删除使用 DELETE 请求，同样，需要根据 id 进行删除：

**语法：**

```javascript
DELETE /{索引库名}/_doc/id值
```

**示例：**

```json
DELETE /heima/_doc/1
```

**结果：**

![](微服务常用组件ES-上/H2zYbHVTFol0jWxFohOcZdWbnnh.png)

### 3.4.修改文档

修改有两种方式：

- 全量修改：直接覆盖原来的文档
- 局部修改：修改文档中的部分字段

#### 3.4.1.全量修改

全量修改是覆盖原来的文档，其本质是两步操作：

- 根据指定的 id 删除文档
- 新增一个相同 id 的文档

**注意**：如果根据 id 删除时，id 不存在，第二步的新增也会执行，也就从修改变成了新增操作了。

**语法：**

```json
PUT /{索引库名}/_doc/文档id
{
    "字段1": "值1",
    "字段2": "值2",
    // ... 略
}
```

**示例：**

```json
PUT /heima/_doc/1
{
    "info": "黑马程序员高级Java讲师",
    "email": "zy@itcast.cn",
    "name": {
        "firstName": "云",
        "lastName": "赵"
    }
}
```

由于 `id` 为 `1` 的文档已经被删除，所以第一次执行时，得到的反馈是 `created`：

![](微服务常用组件ES-上/W9uvbn4KyoKuL8xnIWocxMUqnnd.png)

所以如果执行第 2 次时，得到的反馈则是 `updated`：

![](微服务常用组件ES-上/WgoBbqTxloZJKyxYzgFcy4dYnTc.png)

#### 3.4.2.局部修改

局部修改是只修改指定 id 匹配的文档中的部分字段。

**语法：**

```json
POST /{索引库名}/_update/文档id
{
    "doc": {
         "字段名": "新的值",
    }
}
```

**示例：**

```json
POST /heima/_update/1
{
  "doc": {
    "email": "ZhaoYun@itcast.cn"
  }
}
```

**执行结果**：

![](微服务常用组件ES-上/WRKaboLO8oKfIDxA5tpcOYXMnTc.png)

### 3.5.批处理

批处理采用 POST 请求，基本语法如下：

```java
POST _bulk
{ "index" : { "_index" : "test", "_id" : "1" } }
{ "field1" : "value1" }
{ "delete" : { "_index" : "test", "_id" : "2" } }
{ "create" : { "_index" : "test", "_id" : "3" } }
{ "field1" : "value3" }
{ "update" : {"_id" : "1", "_index" : "test"} }
{ "doc" : {"field2" : "value2"} }
```

其中：

- `index` 代表新增操作

  - `_index`：指定索引库名
  - `_id` 指定要操作的文档 id
  - `{ "field1" : "value1" }`：则是要新增的文档内容
- `delete` 代表删除操作

  - `_index`：指定索引库名
  - `_id` 指定要操作的文档 id
- `update` 代表更新操作

  - `_index`：指定索引库名
  - `_id` 指定要操作的文档 id
  - `{ "doc" : {"field2" : "value2"} }`：要更新的文档字段

示例，批量新增：

```java
POST /_bulk
{"index": {"_index":"heima", "_id": "3"}}
{"info": "黑马程序员C++讲师", "email": "ww@itcast.cn", "name":{"firstName": "五", "lastName":"王"}}
{"index": {"_index":"heima", "_id": "4"}}
{"info": "黑马程序员前端讲师", "email": "zhangsan@itcast.cn", "name":{"firstName": "三", "lastName":"张"}}
```

批量删除：

```java
POST /_bulk
{"delete":{"_index":"heima", "_id": "3"}}
{"delete":{"_index":"heima", "_id": "4"}}
```

### 3.6.总结

文档操作有哪些？

- 创建文档：`POST /{索引库名}/_doc/文档id   { json文档 }`
- 查询文档：`GET /{索引库名}/_doc/文档id`
- 删除文档：`DELETE /{索引库名}/_doc/文档id`
- 修改文档：

  - 全量修改：`PUT /{索引库名}/_doc/文档id { json文档 }`
  - 局部修改：`POST /{索引库名}/_update/文档id { "doc": {字段}}`

## 4.RestAPI

ES 官方提供了各种不同语言的客户端，用来操作 ES。这些客户端的本质就是组装 DSL 语句，通过 http 请求发送给 ES。

官方文档地址：

[https://www.elastic.co/guide/en/elasticsearch/client/index.html](https://www.elastic.co/guide/en/elasticsearch/client/index.html)

由于 ES 目前最新版本是 8.8，提供了全新版本的客户端，老版本的客户端已经被标记为过时。而我们采用的是 7.12 版本，因此只能使用老版本客户端：

![](微服务常用组件ES-上/SEwPbjRjUowUxtxhx9KczlnlnZe.png)

然后选择 7.12 版本，HighLevelRestClient 版本：

![](微服务常用组件ES-上/Hec1bwCnPoHU2axtOAFcefhvnHe.png)

### 4.1.初始化 RestClient

在 elasticsearch 提供的 API 中，与 elasticsearch 一切交互都封装在一个名为 `RestHighLevelClient` 的类中，必须先完成这个对象的初始化，建立与 elasticsearch 的连接。

分为三步：

1）在 `item-service` 模块中引入 `es` 的 `RestHighLevelClient` 依赖：

```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
</dependency>
```

2）因为 SpringBoot 默认的 ES 版本是 `7.17.10`，所以我们需要覆盖默认的 ES 版本：

```xml
<properties>
      <maven.compiler.source>11</maven.compiler.source>
      <maven.compiler.target>11</maven.compiler.target>
      <elasticsearch.version>7.12.1</elasticsearch.version>
  </properties>
```

3）初始化 RestHighLevelClient：

初始化的代码如下：

```java
RestHighLevelClient client = new RestHighLevelClient(RestClient.builder(
        HttpHost.create("http://192.168.150.101:9200")
));
```

这里为了单元测试方便，我们创建一个测试类 `IndexTest`，然后将初始化的代码编写在 `@BeforeEach` 方法中：

```java
package com.hmall.item.es;

import org.apache.http.HttpHost;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import java.io.IOException;

public class IndexTest {

    private RestHighLevelClient client;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(RestClient.builder(
                HttpHost.create("http://192.168.150.101:9200")
        ));
    }

    @Test
    void testConnect() {
        System.out.println(client);
    }

    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }
}
```

### 4.1.创建索引库

由于要实现对商品搜索，所以我们需要将商品添加到 Elasticsearch 中，不过需要根据搜索业务的需求来设定索引库结构，而不是一股脑的把 MySQL 数据写入 Elasticsearch.

#### 4.1.1.Mapping 映射

搜索页面的效果如图所示：

![](微服务常用组件ES-上/MAhTbzSuOohtFQxIjcKcqaUInZf.png)

实现搜索功能需要的字段包括三大部分：

- 搜索过滤字段

  - 分类
  - 品牌
  - 价格
- 排序字段

  - 默认：按照更新时间降序排序
  - 销量
  - 价格
- 展示字段

  - 商品 id：用于点击后跳转
  - 图片地址
  - 是否是广告推广商品
  - 名称
  - 价格
  - 评价数量
  - 销量

对应的商品表结构如下，索引库无关字段已经划掉：

![](微服务常用组件ES-上/Q8AkbdspSoKBWmxztmucTOzJnrf.png)

结合数据库表结构，以上字段对应的 mapping 映射属性如下：

| **字段名**   |      | **字段类型** | **类型说明**           | **是否****参与搜索** | **是否****参与分词** | **分词器** |
| ------------ | ---- | ------------ | ---------------------- | -------------------- | -------------------- | ---------- |
| id           |      | `long`       | 长整数                 | - [x]                | - [ ]                | ——         |
| name         |      | `text`       | 字符串，参与分词搜索   | - [x]                | - [x]                | IK         |
| price        |      | `integer`    | 以分为单位，所以是整数 | - [x]                | - [ ]                | ——         |
| stock        |      | `integer`    | 字符串，但需要分词     | - [x]                | - [ ]                | ——         |
| image        |      | `keyword`    | 字符串，但是不分词     | - [ ]                | - [ ]                | ——         |
| category     |      | `keyword`    | 字符串，但是不分词     | - [x]                | - [ ]                | ——         |
| brand        |      | `keyword`    | 字符串，但是不分词     | - [x]                | - [ ]                | ——         |
| sold         |      | `integer`    | 销量，整数             | - [x]                | - [ ]                | ——         |
| commentCount |      | `integer`    | 评价，整数             | - [ ]                | - [ ]                | ——         |
| isAD         |      | `boolean`    | 布尔类型               | - [x]                | - [ ]                | ——         |
| updateTime   |      | `Date`       | 更新时间               | - [x]                | - [ ]                | ——         |

因此，最终我们的索引库文档结构应该是这样：

```json
PUT /items
{
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword"
      },
      "name":{
        "type": "text",
        "analyzer": "ik_max_word"
      },
      "price":{
        "type": "integer"
      },
      "stock":{
        "type": "integer"
      },
      "image":{
        "type": "keyword",
        "index": false
      },
      "category":{
        "type": "keyword"
      },
      "brand":{
        "type": "keyword"
      },
      "sold":{
        "type": "integer"
      },
      "commentCount":{
        "type": "integer",
        "index": false
      },
      "isAD":{
        "type": "boolean"
      },
      "updateTime":{
        "type": "date"
      }
    }
  }
}
```

#### 4.1.2.创建索引

创建索引库的 API 如下：

![](微服务常用组件ES-上/LXD6brggXoeg3WxW5ztc6IBrnSe.jpeg)

代码分为三步：

- 1）创建 Request 对象。

  - 因为是创建索引库的操作，因此 Request 是 `CreateIndexRequest`。
- 2）添加请求参数

  - 其实就是 Json 格式的 Mapping 映射参数。因为 json 字符串很长，这里是定义了静态字符串常量 `MAPPING_TEMPLATE`，让代码看起来更加优雅。
- 3）发送请求

  - `client.indices()` 方法的返回值是 `IndicesClient` 类型，封装了所有与索引库操作有关的方法。例如创建索引、删除索引、判断索引是否存在等

在 `item-service` 中的 `IndexTest` 测试类中，具体代码如下：

```java
@Test
void testCreateIndex() throws IOException {
    // 1.创建Request对象
    CreateIndexRequest request = new CreateIndexRequest("items");
    // 2.准备请求参数
    request.source(MAPPING_TEMPLATE, XContentType.JSON);
    // 3.发送请求
    client.indices().create(request, RequestOptions.DEFAULT);
}

static final String MAPPING_TEMPLATE = "{\n" +
            "  \"mappings\": {\n" +
            "    \"properties\": {\n" +
            "      \"id\": {\n" +
            "        \"type\": \"keyword\"\n" +
            "      },\n" +
            "      \"name\":{\n" +
            "        \"type\": \"text\",\n" +
            "        \"analyzer\": \"ik_max_word\"\n" +
            "      },\n" +
            "      \"price\":{\n" +
            "        \"type\": \"integer\"\n" +
            "      },\n" +
            "      \"stock\":{\n" +
            "        \"type\": \"integer\"\n" +
            "      },\n" +
            "      \"image\":{\n" +
            "        \"type\": \"keyword\",\n" +
            "        \"index\": false\n" +
            "      },\n" +
            "      \"category\":{\n" +
            "        \"type\": \"keyword\"\n" +
            "      },\n" +
            "      \"brand\":{\n" +
            "        \"type\": \"keyword\"\n" +
            "      },\n" +
            "      \"sold\":{\n" +
            "        \"type\": \"integer\"\n" +
            "      },\n" +
            "      \"commentCount\":{\n" +
            "        \"type\": \"integer\"\n" +
            "      },\n" +
            "      \"isAD\":{\n" +
            "        \"type\": \"boolean\"\n" +
            "      },\n" +
            "      \"updateTime\":{\n" +
            "        \"type\": \"date\"\n" +
            "      }\n" +
            "    }\n" +
            "  }\n" +
            "}";
```

### 4.2.删除索引库

删除索引库的请求非常简单：

```json
DELETE /hotel
```

与创建索引库相比：

- 请求方式从 PUT 变为 DELTE
- 请求路径不变
- 无请求参数

所以代码的差异，注意体现在 Request 对象上。流程如下：

- 1）创建 Request 对象。这次是 DeleteIndexRequest 对象
- 2）准备参数。这里是无参，因此省略
- 3）发送请求。改用 delete 方法

在 `item-service` 中的 `IndexTest` 测试类中，编写单元测试，实现删除索引：

```java
@Test
void testDeleteIndex() throws IOException {
    // 1.创建Request对象
    DeleteIndexRequest request = new DeleteIndexRequest("items");
    // 2.发送请求
    client.indices().delete(request, RequestOptions.DEFAULT);
}
```

### 4.3.判断索引库是否存在

判断索引库是否存在，本质就是查询，对应的请求语句是：

```json
GET /hotel
```

因此与删除的 Java 代码流程是类似的，流程如下：

- 1）创建 Request 对象。这次是 GetIndexRequest 对象
- 2）准备参数。这里是无参，直接省略
- 3）发送请求。改用 exists 方法

```java
@Test
void testExistsIndex() throws IOException {
    // 1.创建Request对象
    GetIndexRequest request = new GetIndexRequest("items");
    // 2.发送请求
    boolean exists = client.indices().exists(request, RequestOptions.DEFAULT);
    // 3.输出
    System.err.println(exists ? "索引库已经存在！" : "索引库不存在！");
}
```

### 4.4.总结

JavaRestClient 操作 elasticsearch 的流程基本类似。核心是 `client.indices()` 方法来获取索引库的操作对象。

索引库操作的基本步骤：

- 初始化 `RestHighLevelClient`
- 创建 XxxIndexRequest。XXX 是 `Create`、`Get`、`Delete`
- 准备请求参数（ `Create` 时需要，其它是无参，可以省略）
- 发送请求。调用 `RestHighLevelClient#indices().xxx()` 方法，xxx 是 `create`、`exists`、`delete`

## 5.RestClient 操作文档

索引库准备好以后，就可以操作文档了。为了与索引库操作分离，我们再次创建一个测试类，做两件事情：

- 初始化 RestHighLevelClient
- 我们的商品数据在数据库，需要利用 IHotelService 去查询，所以注入这个接口

```java
package com.hmall.item.es;

import com.hmall.item.service.IItemService;
import org.apache.http.HttpHost;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestHighLevelClient;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeEach;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.io.IOException;

@SpringBootTest(properties = "spring.profiles.active=local")
public class DocumentTest {

    private RestHighLevelClient client;
    @Autowired
    private IItemService itemService;

    @BeforeEach
    void setUp() {
        this.client = new RestHighLevelClient(RestClient.builder(
                HttpHost.create("http://192.168.150.101:9200")
        ));
    }
    
    @AfterEach
    void tearDown() throws IOException {
        this.client.close();
    }
}
```

### 5.1.新增文档

我们需要将数据库中的商品信息导入 elasticsearch 中，而不是造假数据了。

#### 5.1.1.实体类

索引库结构与数据库结构还存在一些差异，因此我们要定义一个索引库结构对应的实体。

在 `hm-service` 模块的 `com.hmall.item.domain.dto` 包中定义一个新的 DTO：

```java
package com.hmall.item.domain.po;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

import java.time.LocalDateTime;

@Data
@ApiModel(description = "索引库实体")
public class ItemDoc{

    @ApiModelProperty("商品id")
    private String id;

    @ApiModelProperty("商品名称")
    private String name;

    @ApiModelProperty("价格（分）")
    private Integer price;

    @ApiModelProperty("商品图片")
    private String image;

    @ApiModelProperty("类目名称")
    private String category;

    @ApiModelProperty("品牌名称")
    private String brand;

    @ApiModelProperty("销量")
    private Integer sold;

    @ApiModelProperty("评论数")
    private Integer commentCount;

    @ApiModelProperty("是否是推广广告，true/false")
    private Boolean isAD;

    @ApiModelProperty("更新时间")
    private LocalDateTime updateTime;
}
```

#### 5.1.2.API 语法

新增文档的请求语法如下：

```json
POST /{索引库名}/_doc/1
{
    "name": "Jack",
    "age": 21
}
```

对应的 JavaAPI 如下：

![](微服务常用组件ES-上/NpdLbaq5zoKcIhxdmmPccAuPnWg.png)

可以看到与索引库操作的 API 非常类似，同样是三步走：

- 1）创建 Request 对象，这里是 `IndexRequest`，因为添加文档就是创建倒排索引的过程
- 2）准备请求参数，本例中就是 Json 文档
- 3）发送请求

变化的地方在于，这里直接使用 `client.xxx()` 的 API，不再需要 `client.indices()` 了。

#### 5.1.3.完整代码

我们导入商品数据，除了参考 API 模板“三步走”以外，还需要做几点准备工作：

- 商品数据来自于数据库，我们需要先查询出来，得到 `Item` 对象
- `Item` 对象需要转为 `ItemDoc` 对象
- `ItemDTO` 需要序列化为 `json` 格式

因此，代码整体步骤如下：

- 1）根据 id 查询商品数据 `Item`
- 2）将 `Item` 封装为 `ItemDoc`
- 3）将 `ItemDoc` 序列化为 JSON
- 4）创建 IndexRequest，指定索引库名和 id
- 5）准备请求参数，也就是 JSON 文档
- 6）发送请求

在 `item-service` 的 `DocumentTest` 测试类中，编写单元测试：

```java
@Test
void testAddDocument() throws IOException {
    // 1.根据id查询商品数据
    Item item = itemService.getById(100002644680L);
    // 2.转换为文档类型
    ItemDoc itemDoc = BeanUtil.copyProperties(item, ItemDoc.class);
    // 3.将ItemDTO转json
    String doc = JSONUtil.toJsonStr(itemDoc);

    // 1.准备Request对象
    IndexRequest request = new IndexRequest("items").id(itemDoc.getId());
    // 2.准备Json文档
    request.source(doc, XContentType.JSON);
    // 3.发送请求
    client.index(request, RequestOptions.DEFAULT);
}
```

### 5.2.查询文档

我们以根据 id 查询文档为例

#### 5.2.1.语法说明

查询的请求语句如下：

```json
GET /{索引库名}/_doc/{id}
```

与之前的流程类似，代码大概分 2 步：

- 创建 Request 对象
- ~~准备请求参数，这里是无参，直接省略~~
- 发送请求

不过查询的目的是得到结果，解析为 ItemDTO，还要再加一步对结果的解析。示例代码如下：

![](微服务常用组件ES-上/EkqjbfS1JocZSfxG8excksdqnYb.png)

可以看到，响应结果是一个 JSON，其中文档放在一个 `_source` 属性中，因此解析就是拿到 `_source`，反序列化为 Java 对象即可。

其它代码与之前类似，流程如下：

- 1）准备 Request 对象。这次是查询，所以是 `GetRequest`
- 2）发送请求，得到结果。因为是查询，这里调用 `client.get()` 方法
- 3）解析结果，就是对 JSON 做反序列化

#### 5.2.2.完整代码

在 `item-service` 的 `DocumentTest` 测试类中，编写单元测试：

```java
@Test
void testGetDocumentById() throws IOException {
    // 1.准备Request对象
    GetRequest request = new GetRequest("items").id("100002644680");
    // 2.发送请求
    GetResponse response = client.get(request, RequestOptions.DEFAULT);
    // 3.获取响应结果中的source
    String json = response.getSourceAsString();
    
    ItemDoc itemDoc = JSONUtil.toBean(json, ItemDoc.class);
    System.out.println("itemDoc= " + ItemDoc);
}
```

### 5.3.删除文档

删除的请求语句如下：

```json
DELETE /hotel/_doc/{id}
```

与查询相比，仅仅是请求方式从 `DELETE` 变成 `GET`，可以想象 Java 代码应该依然是 2 步走：

- 1）准备 Request 对象，因为是删除，这次是 `DeleteRequest` 对象。要指定索引库名和 id
- 2）~~准备参数，无参，直接省略~~
- 3）发送请求。因为是删除，所以是 `client.delete()` 方法

在 `item-service` 的 `DocumentTest` 测试类中，编写单元测试：

```java
@Test
void testDeleteDocument() throws IOException {
    // 1.准备Request，两个参数，第一个是索引库名，第二个是文档id
    DeleteRequest request = new DeleteRequest("item", "100002644680");
    // 2.发送请求
    client.delete(request, RequestOptions.DEFAULT);
}
```

### 5.4.修改文档

修改我们讲过两种方式：

- 全量修改：本质是先根据 id 删除，再新增
- 局部修改：修改文档中的指定字段值

在 RestClient 的 API 中，全量修改与新增的 API 完全一致，判断依据是 ID：

- 如果新增时，ID 已经存在，则修改
- 如果新增时，ID 不存在，则新增

这里不再赘述，我们主要关注局部修改的 API 即可。

#### 5.4.1.语法说明

局部修改的请求语法如下：

```json
POST /{索引库名}/_update/{id}
{
  "doc": {
    "字段名": "字段值",
    "字段名": "字段值"
  }
}
```

代码示例如图：

![](微服务常用组件ES-上/O9bgbYBPooOpcmxhFk0c9XP2n7f.png)

与之前类似，也是三步走：

- 1）准备 `Request` 对象。这次是修改，所以是 `UpdateRequest`
- 2）准备参数。也就是 JSON 文档，里面包含要修改的字段
- 3）更新文档。这里调用 `client.update()` 方法

#### 5.4.2.完整代码

在 `item-service` 的 `DocumentTest` 测试类中，编写单元测试：

```java
@Test
void testUpdateDocument() throws IOException {
    // 1.准备Request
    UpdateRequest request = new UpdateRequest("items", "100002644680");
    // 2.准备请求参数
    request.doc(
            "price", 58800,
            "commentCount", 1
    );
    // 3.发送请求
    client.update(request, RequestOptions.DEFAULT);
}
```

### 5.5.批量导入文档

在之前的案例中，我们都是操作单个文档。而数据库中的商品数据实际会达到数十万条，某些项目中可能达到数百万条。

我们如果要将这些数据导入索引库，肯定不能逐条导入，而是采用批处理方案。常见的方案有：

- 利用 Logstash 批量导入

  - 需要安装 Logstash
  - 对数据的再加工能力较弱
  - 无需编码，但要学习编写 Logstash 导入配置
- 利用 JavaAPI 批量导入

  - 需要编码，但基于 JavaAPI，学习成本低
  - 更加灵活，可以任意对数据做再加工处理后写入索引库

接下来，我们就学习下如何利用 JavaAPI 实现批量文档导入。

#### 5.5.1.语法说明

批处理与前面讲的文档的 CRUD 步骤基本一致：

- 创建 Request，但这次用的是 `BulkRequest`
- 准备请求参数
- 发送请求，这次要用到 `client.bulk()` 方法

`BulkRequest` 本身其实并没有请求参数，其本质就是将多个普通的 CRUD 请求组合在一起发送。例如：

- 批量新增文档，就是给每个文档创建一个 `IndexRequest` 请求，然后封装到 `BulkRequest` 中，一起发出。
- 批量删除，就是创建 N 个 `DeleteRequest` 请求，然后封装到 `BulkRequest`，一起发出

因此 `BulkRequest` 中提供了 `add` 方法，用以添加其它 CRUD 的请求：

![](微服务常用组件ES-上/OMpfbCns7oWY30xQfdmc07kon1c.png)

可以看到，能添加的请求有：

- `IndexRequest`，也就是新增
- `UpdateRequest`，也就是修改
- `DeleteRequest`，也就是删除

因此 Bulk 中添加了多个 `IndexRequest`，就是批量新增功能了。示例：

```java
@Test
void testBulk() throws IOException {
    // 1.创建Request
    BulkRequest request = new BulkRequest();
    // 2.准备请求参数
    request.add(new IndexRequest("items").id("1").source("json doc1", XContentType.JSON));
    request.add(new IndexRequest("items").id("2").source("json doc2", XContentType.JSON));
    // 3.发送请求
    client.bulk(request, RequestOptions.DEFAULT);
}
```

#### 5.5.2.完整代码

当我们要导入商品数据时，由于商品数量达到数十万，因此不可能一次性全部导入。建议采用循环遍历方式，每次导入 1000 条左右的数据。

`item-service` 的 `DocumentTest` 测试类中，编写单元测试：

```java
@Test
void testLoadItemDocs() throws IOException {
    // 分页查询商品数据
    int pageNo = 1;
    int size = 1000;
    while (true) {
        Page<Item> page = itemService.lambdaQuery().eq(Item::getStatus, 1).page(new Page<Item>(pageNo, size));
        // 非空校验
        List<Item> items = page.getRecords();
        if (CollUtils.isEmpty(items)) {
            return;
        }
        log.info("加载第{}页数据，共{}条", pageNo, items.size());
        // 1.创建Request
        BulkRequest request = new BulkRequest("items");
        // 2.准备参数，添加多个新增的Request
        for (Item item : items) {
            // 2.1.转换为文档类型ItemDTO
            ItemDoc itemDoc = BeanUtil.copyProperties(item, ItemDoc.class);
            // 2.2.创建新增文档的Request对象
            request.add(new IndexRequest()
                            .id(itemDoc.getId())
                            .source(JSONUtil.toJsonStr(itemDoc), XContentType.JSON));
        }
        // 3.发送请求
        client.bulk(request, RequestOptions.DEFAULT);

        // 翻页
        pageNo++;
    }
}
```

### 5.6.小结

文档操作的基本步骤：

- 初始化 `RestHighLevelClient`
- 创建 XxxRequest。

  - XXX 是 `Index`、`Get`、`Update`、`Delete`、`Bulk`
- 准备参数（`Index`、`Update`、`Bulk` 时需要）
- 发送请求。

  - 调用 `RestHighLevelClient#.xxx()` 方法，xxx 是 `index`、`get`、`update`、`delete`、`bulk`
- 解析结果（`Get` 时需要）

## 6.作业

### 6.1.服务拆分

搜索业务并发压力可能会比较高，目前与商品服务在一起，不方便后期优化。

**需求**：创建一个新的微服务，命名为 `search-service`，将搜索相关功能抽取到这个微服务中

### 6.2.商品查询接口

在 `item-service` 服务中提供一个根据 id 查询商品的功能，并编写对应的 FeignClient

### 6.3.数据同步

每当商品服务对商品实现增删改时，索引库的数据也需要同步更新。

**提示**：可以考虑采用 MQ 异步通知实现。
