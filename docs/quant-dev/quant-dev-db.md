---
title: Quant Dev 4 - 数据库
tags: [Hedge Fund, Quant, Python，Database, SQL]
categories: Career
date: 2022-10-03
---

继续之前的系列，这一期我们讲讲 Quant Dev 经常会接触到的数据库。同样，这里我们不讨论高频交易，虽然决大部分内容都是重合的。

数据库的类型和优缺点本身是一个非常大的主题，我无法也没有能力在一篇文章中阐述清楚，这里仅介绍一些量化领域常见的数据库实现和使用场景。

应该注意的是，通用数据库不一定适合某些量化交易使用场景，因此很多公司在使用通用数据库的同时也会开发内部的专用数据可来解决特定的问题。

## 前置信息

分析数据库，我们从以下几个方面展开：

- 数据模型，Data Model
    - set of records，row based
    - graph
    - document，json，bson ...
    - columnar
- 数据类型描述，Data Define Language - Schema
- 查询语言，Manipulation and Query Language
    - SQL
    - awk
    - 通用语言，c,python,java,etc..
- 水平、纵向拓展模型

量化交易涉及到的数据库可以分成几类：

- 定制数据库
    - 完全定制，比如
        - [Next generation of Arctic](https://github.com/man-group/arctic)
    - 半定制，比如
        - [Man Arctic](https://github.com/man-group/arctic)
- 通用数据库
    - 有固定 Schema 的
        - 关系型 Relational，比如 PostgreSQL，MySQL，TimescaleDB
        - 列数据库 Columnar，比如 KDB，Cassandra
    - 无固定 Schema 的
        - 文档型 Document，比如 MongoDB，InfluxDB

值得注意的是，很多时候定制数据库也不需要重新设计并实现一套数据库（存储、执行、优化等等），我们可以使用已有的数据库进行二次开发。比如，Arctic 就是一个基于 MongoDB 的时间序列数据库，而 TimescaleDB 就是基于 PostgreSQL 开发的时间序列数据库。

另外，在分析数据库的时候还需要注意使用场景：

- OLTP：适合高写入、高读取，但是数据量较小的场景
- OLAP：适合低写入，但是分析复杂且数据量大的场景

![](https://i.imgur.com/aCsAYa3.png)

## 我们需要什么技能

*SQL 语言是必要的。*

因为 SQL 的使用场景已经超出了关系型数据库，比如列数据库 Cassandra 的查询语言 CQL 其实也是类 SQL 语言；TimescaleDB 是时间序列数据可，其查询语言也是 SQL；即使是文档型数据库 也可以使用 SQL 进行查询。

另一方面，SQL 是一个设计非常优秀的描述性计算机语言，识别适合描述数据逻辑，由于不需要指定如何进行查询，数据库后台根据查询语句进行进行充分的优化。

最后，SQL 也是很多 DataFrame API（如 Pandas）的设计蓝本。

*PostgreSQL*

选择一个 SQL 数据库，学习基本的 SQL 概念，比如Schema、表、View、Trigger、约束等等。至于使用那个一实现，可根据公司的情况，不知道用哪个就选择 PostgreSQL，它是最原汁原味的开源 SQL 数据库。

*MongoDB*

文档型数据库的使用，基本的增删查改操作，Index的使用等等。

## 选择数据库

量化行业绝大部分数据可以直接使用 SQL 数据库进行存储和查询。比如中低频的价格信息、标的物的参照信息、标的物的元数据（mapping、id等等）、财报数据等等。

量化行业的另一类数据就是时间序列了。这类信息也可以采用 SQL 数据库存储，但是由于这种信息通常是追加为主，很少修改，而且查询通常涉及非常多的历史记录，且比较复杂，一般会采用专门的数据库或者数据库前端进行处理。比如 TimescaleDB 或者 Arctic。