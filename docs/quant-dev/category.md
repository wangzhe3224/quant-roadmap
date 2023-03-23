---
date: 2021-11-06
categories: Career
tags: [quant-dev, hedge-fund]
comments: true
status: new
---

!!! note
    视频讲解：

    - [油管](https://www.youtube.com/watch?v=O9gK161q_HA&list=PL5ETbHWvsj-FJUbOyDHHqu6IyK6Sd17Ba&index=7)
    - [B站](https://www.bilibili.com/video/BV1E44y1v7Zp/?spm_id_from=333.999.0.0)
    - [知乎](https://www.zhihu.com/zvideo/1443293377500045312)

量化对冲基金程序员分类：

- 基础设施
- 运维
- 业务 (Quant Dev)
    - 基础设施
    - 交易系统
    - 研究系统
    - 量化开发
    - 前端开发
  
## 基础设施，Infrastructure

1. 主要职能

基础设施的程序员主要是负责公司的整体计算机基础设施，保证计算、存储和网络资源正常运行，满足业务需求。

包括（不是全部）：

- 数据、计算中心硬件的维护和升级
- 计算资源集群，Spark集群，k8s集群，日常的虚拟机群等等
- 网络，网络安全、VPN等等
- 分布式文件系统
- 数据库开发和维护

2. 技术栈

- Linux
- k8s
- Spark
- 数据库

3. 语言

- C/C++/Rust
- Bash

## 运维，Dev Ops

1. 主要职能

对冲基金的运维，跟其他科技公司的运维功能类似，主要是确保各个应用的连续测试和构建，合理化的容器化APP等等。

2. 技术栈

- Linux
- Jenkins
- K8s
- Docker

3. 语言

- Bash
- Python

## 业务，Quant Dev

这部分是量化对冲基金比较独有的程序员，每个公司可能也会不太一样，这部分程序员需要对金融知识，
不过大体可以分为：

- 基础设施
- 交易系统
- 研究系统
- 量化开发
- 前端开发

### 基础设施

这部分的基础设施主要是面向业务逻辑的一些工具和框架的开发，比如

- 高性能时间序列数据库
- 实时计算图，Computation Graph
- 机器学习框架
- 股票的话，比如Ticker map，分红，换名字啊等等一些琐碎但是重要的事情

语言框架：

- C++/Rust/Python

### 交易系统

负责订单的执行和仓位管理，主要是负责把策略生成的信号或者订单提交到不同的Broker进行执行，并且
返回执行结果，也负责仓位的查询等等。这块其实内容很丰富，也会直接影响公司的PnL。

语言框架：

- Java/Python

### 研究系统

这部分主要是为了研究人员进行量化研究做一些工具，比如回测工具，基本的分析模块，新的数据集，
Alpha的分析等。

语言框架：

- Python

### 量化开发

这部分程序员的主要任务是把研究人员证明可行的信号，转化成可以实际交易执行的代码，投放到交易系统
进行交易。属于连接想法和实践的桥梁，这块其实内容很丰富，也会直接影响公司的PnL。

语言框架：

- Java/Python

### 前端开发

这部分跟正常的前端差不多，主要为后端API提供一个GUI

语言框架：

- Js/Python
- React
