---
title: 股票中低频股票系统化交易 | Quant dev 技能框架
tags: Quant
categories: Career
date: 2022-03-08
math: true
---

中低频策略主要由两个方面的定义：

1. 持仓周期以天为单位（注意是持仓周期，不是交易频率，中低频也存在日内交易）
2. 目标仓位生成与交易执行之间的反馈循环频率较低

- [场景](#场景)
  - [研究回测](#研究回测)
  - [实盘交易](#实盘交易)
  - [复盘审核](#复盘审核)
- [基础组成部分](#基础组成部分)
- [软件框架](#软件框架)
  - [数据存储和版本控制](#数据存储和版本控制)
  - [计算](#计算)
  - [实盘交易](#实盘交易-1)
- [系统实现](#系统实现)
  - [数据](#数据)
  - [计算](#计算-1)
  - [部署](#部署)
- [软件和框架总结](#软件和框架总结)

## 场景

系统化交易的实际场景主要可以分为：研究回测、实盘交易和复盘审核。因为产生目标投资组合（Target）与订单执行（Execution）的反馈循环较弱，通常可以拆分目标投资组合生成和订单执行，这是中低频策略的主要特征之一。

设计系统化交易系统主要实现如下目标：

- 可回溯，包括数据和配置
- 可警报
- 人工干预接口
- 易于拓展，包括数据、信号和附加组件

系统化交易中所有信息都是应该是时间序列，且是 Point in time，PIT 的。换句话说，每一个数据点应该有两个时间戳：事件发生的时间（$t$, time）和事件被观测的时间（$ot$, observe time）。

另外一个思路就是 Append Only，即只增加，不更新。这样每个事件只有一个时间，即发生的时间，$t$。但是，我们需要重构历史，或者建立快照，snapshot，来提速。

这样做的主要原因是提供了回溯的可能性，而可回溯是盘后分析、模拟实盘差异分析的关键。

### 研究回测

这个部分也可以叫做**信号生成**。

这部分的产出是某个时间节点，$t$，的目标投资组合 $sig_t$ 或者 $signal_t$，即在该时间点，我们希望实现的持仓，通常是一揽子股票、期货的权重，$w_i$，即 $sig := {w_0, w_1, ..., w_N}$，其中 $i$ 代表交易标的物。

通常，目标投资组合是历史数据的函数，$sig_t = g(data_{0..t-1}, config_t)$。这里 $data$ 可以包含各种类型，比如常见的价格、成交量数据、公司财报、天气、专家意见、Stock Loan $sl$、持仓 $pos$、交易约束 $cons$ 等等。$config_t$ 属于策略的配置部分，这部分就是系统化交易的人工因素。

当然，为了评估策略的历史表现，我们需要对信号，即 ${sig_0, sig_1, ..., sig_n}$，的历史收益情况进行计算，$PnL = f(sig, price, cost)$，即**回测**。$PnL$ 通常是信号、标的价格、花费的函数。在这个部分，$cost$ 的计算比较复杂，可以建立各种模型，也是研究回测的重头戏，**花费的计算对评估策略的表现有至关重要的作用**。

### 实盘交易

理想情况下，实盘交易只需要生成最新的目标，$sig_t$，当然这不代表我们只需要 $data_{t-1}$。事实上，我们希望实盘交易系统和回测研究系统采用相同的，$g$ 和 $data$ 来计算。这一点在设计交易系统时非常重要。虽然，$g$ 是一样的，但是通常实盘仅仅需要近期的历史数据。

实盘交易过程中还有一个重要的部分，就是审核。审核包含：数据异常、信号异常、订单异常三个主要部分。审核的主要目的是确保每个时间点，$t$，发出的订单都是正确和合理的。

最后，在实盘交易中还存在很大的操作风险，即 Operation Risk。这部分其实跟审核相关，也跟运行时异常相关。比如，数据出现异常，导致信号异常，如何人工介入重写？

$sig_{new} = h(sig_t, data, human)$

其中，$human$ 属于主观部分。

实盘最后一步就是根据当前仓位和信号生成订单提交：

$order_t = pos_{t-1} - sig_{t}$ 

### 复盘审核

复盘这个部分主要是计算回测结果和实盘结果之间的差异，并寻找差异根源。模拟和实盘的差别可以说是系统化交易一个老大难问题了。

$\Delta_t^{sig} = sig_t^{sim} - sig_t^{live}$

$\Delta_t^{pnl} = PnL_t^{sim} - PnL_t^{live}$

$\Delta_t^{sig}$ 是模拟得到的投资组合权重与实盘记录的投资组合的权重之间的差异，$\Delta_t^{pnl}$ 是模拟盈亏和实盘盈亏之间的差异。这两个差异是不同的，因为 $PnL^{sim} = f(sig, price, cost)$，而实盘盈亏是直接根据仓位计算的，这里面除了 $sig$ 至少多一个 $cost$ 的区别，而且还可能有其他的区别。

## 基础组成部分

通过上述分析可以发现如下值：

- $data$，一切数据源
- $sig^{sim}$，信号或者 Alpha
- $config$，配置文件
- $human$，人工输入
- $cons$，交易约束
- $cost^{sim}$，模拟交易成本
- $pos$，仓位
- $sig^{live}$，实际成交的仓位
- $PnL^{live}$，实盘盈亏
- $cost^{live}$，实盘交易成本

其中，$pos$, $sig^{live}$, $PnL^{live}$, $cost^{live}$ 只能通过实盘记录累积获得历史。

可以发现如下关系：

- $g$, $sig_t = g(data_{0..t-1}, config_t, cost, cons)$
- $h$, $sig_{new} = h(sig_t, data, human)$
- $f$, $PnL^{sim} = f(sig, price, cost)$

其中，$g$ 是数据+配置产生信号的函数，$h$ 是人工干预函数，$f$ 就是模拟函数，即计算模拟盈亏的函数。

复盘信息：

- $\Delta_t^{sig}$
- $\Delta_t^{pnl}$

## 软件框架

识别基础数据类型、函数和需求后，就可以考虑构建支持这种系统的软件架构。

### 数据存储和版本控制

数据主要包含两大类：外部数据和内部数据。

外部数据即由第三方提供的数据，比如：

- 价格、成交量数据
- 基本面数据
- Ticker Mapping
- Reference Data
- 其他非传统数据，比如天气、流量、新闻等等

内部数据即本地产生的数据，比如：

- 策略或者信号的配置文件
- 人为干预事件
- 实盘成交数据、税收、交易费用等等

（文本数据可以轻松版本控制，其他格式呢？比如二进制数据，如何版本控制）

所有数据应该尽可能做到版本控制，一方面可以回溯，另一方面可以快速回滚。

### 计算

计算包括：信号生成和回测、复盘分析、实盘信号生成。

这部分主要考虑如何高效组织大规模计算。通常的做法是采用计算图，Computation Graph，进行组织。

### 实盘交易

实盘交易部分主要涉及：

- 警报系统
- 实盘报告系统
- 人工干预接口
- 执行接口

一般来说，中低频交易系统中，执行系统往往呈现黑箱状态，即通过API进行沟通。

## 系统实现

### 数据

- 低频数据：基于文件、Rest API、爬虫
- 高频数据：socket

主要涉及的软件：

- 消息队列：比如 Kafka
- 高速缓存：比如 Redis
- 持久化：比如 MySQL、MongoDB或者其他定制实现
- 数据管道管理：比如 Airflow

特别是对于持久化，选择那种方案，多数取决于数据的使用场景和频率。在股票系统化交易中，数据多数为二维，少数情况为多维数组。最常见的数据结构就是一个二维的 Table like 数据，索引（index）通常是时间，列通常是股票id或者其他任何标签。写入通常是重写、追加、更新，而读取通常需要一些过滤，比如选中某段时间的，某些列，或者一些窗口等操作。

股票交易中还有一个比较复杂的数据就是ID的映射和一些静态信息，比如一只股票在不同的数据源就有用多个ID，比如 Bloomberg，Reuters，ISIN 等等，他们之间映射往往是 PIT 的，即不同的时间观察，会有不同的结果，这对实盘和回测都有较大影响。

### 计算

计算可以分成两个部分：描述和执行。描述一般是通过 DAG，有向无环图完成，即惰性。而执行则是 DAG 的运行时。目前 DAG 的运行时选择很多，后端也很多。

这里推荐一个：Dask。Dask 的运行时后端可以是 K8s，Spark，或者单机多核心。

当然，DAG 中的每一个任务还是执行代码，这部分 Python 生态也非常优秀：

- numpy
- pandas
- scipy
- tensorflow
- polars

注意，将计算的描述和执行分开是非常有用的抽象，因为计算描述在某种程度上可以被版本控制，而且可以最大限度的降低重复计算，提高效率。

不同的任务之间，可以采用类似 Airflow 的软件进行管理。

### 部署

部署涉及：

- 代码部署
- 计算资源部署
- 策略配置文件部署

再部署方面，把代码、计算资源和配置分离，可以最大限度的增加灵活性和可控性。

这部分用到的主要软件：

- Docker
- Git
- Linux
- Jenkins

另外，最好维护三个不同的环境：dev、stage、prod。dev 是主要的开发环境，变化最快嘴不稳定；stage 是主要的测试环境，相对比较稳定；prod是生产环境，只有通过stage测试阶段的 Image、config 才会被部署到prod。

## 软件和框架总结

- Kafka
- Redis
- MySQL | MongoDB
- Airflow
- Git
- Linux
- Jenkins
- Spark
- Docker
- K8s


语言和库：

- Python
    - numpy
    - pandas
    - sicpy
    - dask
    - polars

