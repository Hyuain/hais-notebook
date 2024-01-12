---
title: Software Engineering
date: 2023-04-10 14:34:12
categories:
  - - 计算机
tags:
  - S1
---

.

<!-- more -->

# Software Engineering

## Software Development Lifecycle

### Waterfall Software Development Lifecycle

在瀑布式中，软件开发被分为几个不同的阶段，每个阶段依次进行：需求分析、设计、编写代码、集成、测试、部署

### Requirements

#### Types of Requirements

**功能性需求（Functional Requirements）**：定义系统需要完成什么（What）

**非功能性需求（Non-functional Requirements）**：定义系统完成得多好（How well），有几种类型：

- **Usability**：易用性
- **Performance**：性能，系统对于特定的用户行为在特定的载荷下的响应速度
- **Scalability**：拓展性，系统面对持续增加的数据、功能、用户的适应性
- **Availability**：可用性，在给定的时间点，系统可被用户访问的可能性
- **Reliability**：可靠性，在给定的时间段和预定义的条件下，系统或他的元素正常运作的可能性
- **Maintainability**：可维护性
- **Security**：安全性
- **Compatibility**：兼容性，对老版本兼容、对旧系统兼容

### Unified Modeling Language (UML)

UML 提供了一种标准可视化系统设计的标准方法，其中包含了很多种图表，主要分为静态的 Structure Diagram 和动态的 Behavior Diagram。

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/02-uml-diagram-types.png)

### Design

主要分为 **面向对象（Object-Oriented Design, OOD）** 和 **面向过程（Procedure-Oriented Design, POD）**。

### Testing

#### Unit Testing

> 对每个单元进行独立和分离的测试（与其他的单元分开）

- 假设其他的部分都是正常的
- 使用假函数来代替实际调用其他单元

自动化测试：测试框架（JUnit）+ CI 工具（GitHub Action）

#### Integration Testing

> 将不同的部分集成起来作为一个组来进行测试

#### System Testing

> 确保系统完成了所有需求文档上规定的需求

#### Acceptance Testing

> 终端用户检验系统是否符合验收标准

验收标准（Acceptance Criteria）包括功能、性能、接口质量、软件整体质量、安全性

验收通常包括 Alpha Test、Beta Test 等

### Deployment

#### "Big Bang" deployment

所有用户在同一时间全部迁移到新系统，风险高，不能简单回滚。

#### Rolling Upgrade

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/SE-RollingUpgrade.png)

#### Blue-Green Deployment

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/SE-BlueGreen.png)

#### Canary Deployment

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/SE-Canary.png)

#### Visioned Deloyment

保持所有版本的可用性，API 开发者经常会采取这样的方式，但是需要消耗更多的资源

### Maintenance

软件维护主要有以下几类：

- **Corrective Maintenance**：纠正性维护，纠正发现的问题
- **Adaptive Maintenance**：适应性维护，保持软件在变化的环境中可用
- **Positive Maintenance**：完善维护，交付后对产品进行修改，提高性能或可维护性
- **Preventive Maintenance**：预防性维护，预防潜在的故障

## Scalability

可拓展性指的是调整系统的容量，用性价比高的方式满足需求，处理更多的用户、客户端、数据、事务、请求等。同时需要让向下扩展与向上拓展一样简单。

**Vertical Scaling (Scaling Up)**

- 提升虚拟机的规格
- 不需要对应用做出修改

**Horizontal Scaling (Scaling Out)**

- 一般来讲更加便宜，运行复数个小虚拟机比运行一个大虚拟机更省钱
- 增加新的实例，比如虚拟机或数据库副本

## Availability

$$
Availability = \frac{Uptime}{Uptime + Downtime} = \frac{MTTF}{MTTF+MTTR} = \frac{MTTF}{MTBF}
$$

**MTBF (Mean Time Between Failure)**：两次错误发生之间的时间

**MTTF (Mean Time to Failure)**：错误修好之后到下一次错误之前的时间

**MTTR (Mean Time to Repair)**：修复错误的时间

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/SE-MTTF.png)

系统间的依赖会降低可用性、冗余组件则会增加可用性，比如：

一个系统有两个独立的组件，每个的可用性 99.9%，那么系统的可用性是 100% - 0.1% * 0.1% = 99.99%

## Maintainability

可维护性会影响代码改变的频率

## Microservices

### Scaling Microservices

**Functional Decomposition (Y Axis)**：将功能拆分，让根据每个功能的特点进行拓展

**Horizontal Duplication (X Axis)**：部署多个、独立的实例，通过负载均衡来分配流量

**Data Partitioning (Z Axis)**：将数据通过某种方法分开。可以在数据库维度分开，也可以将数据分到不同的微服务实例中。每个服务器只拿一部分数据。通过 Shading Key 来决定去哪个服务器找我们要的数据。

# Agile Product Ownership



# DevOps

> DevOps = Development ∩ Operation = New Mindset + New Tools + New Skills

DevOps 主要是通过一系列的原则、工具、方法来提高开发与运维的协作，通过快速发布新功能、修复和优化来缩短开发周期。

敏捷开发加速了业务需求 -> 开发，DevOps 则加速了开发 -> 运维。

典型的软件发行流程：想法 -> 需求 -> 编码 -> 测试 -> 打包 -> 部署 -> 运维 & 监控

**DevOps 表现指标：**

- **Delivery Lead Time**：交付前时间，包括代码实现、测试、提交更改到生产系统，越短越好
- **Deployment Frequency**：部署频率，越高越好
- **Change Fail Percentage**：变更导致需要回滚或紧急修复的比率，越低越好
- **Mean Time to Restore (MTTR)**：从突发事件中恢复的平均时间，越短越好

## DevOps Techniques and Tools

### Continuous Integration (CI)

> 持续集成

每个开发人员开发开发自己负责的部分

- 需要持续地将自己的代码与其他的人的代码集成起来
- 需要对集成后的代码进行单元测试、集成测试、回归测试

涉及到以下过程和工具：

- **Source Code Management**：源代码管理，Git、SVN、CVS、ClearCase
- **Build Process**：构建，Maven、Gradle、Rake、JUnit、Sonar
- **Continuous Integration**：持续集成，Jenkins、GitLab CI、Travis、Nexus
- **Test Automation**：自动化测试，包括单元测试、服务测试、UI 测试等

### Continuous Delivery (CD)

> 持续交付并不是说尽可能快地部署所有变更，而是让所有变更在任何时刻都被保证是可部署的。换句话说，CD 是保证在任何时间、任何平台部署任意版本的能力和基础设施。

开发环境和生产环境是不同的

- 开发人员的代码需要经过 Deployment Pipeline，在这个过程中，会经历数个环境，越后面的环境会越接近生产环境
- CD 指的就是持续的让代码经历这些环境，保证新功能在任何时刻都可以扔到生产环境中

工具：Chef、Puppet、Docker、Heroku、Open Stack、AWS Cloud Formation、Rancher、Flyway

#### From CI to CD

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/ShiftLeftInDevOps.png)

#### Continuous Delivery & Continuous Deployment

- **持续部署（Continuous Deployment）是一个过程**。指的是每一个开发人员产生的变化都向生产发布（**Do** deploy to production），他并不一定适用于所有的组织，因为有的组织需要手动发布。
- **持续交付（Continuous Delivery）是一种能力。**指的是在任何时候使用 Pipeline 向不同的环境部署应用的能力（**Can** deploy to production）。拥有这样的能力比真的生产环境连续部署更加重要。

### Microservices

> 微服务

TBC.

### Infrastructure as Code (IaC)

> 基础设置及代码

- 自动化创建、更新、删除设施的流程，简化创建应用所需资源等重复性工作
- 用同一套代码（参数不同）创建不同的环境
  ![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/1*3HV8dJP3XTTGBBMzYwhqlA.png)

### Monitoring and Logging

> 监测与日志，让开发者查看生产环境指标

**Application Performance Monitoring (APM)** 是说监视生产环境的软件，可以先于用户发现问题和需要提升的地方。

*需要在用户发现之前（至少在用户通知你之前）发现问题。*

可用工具：Nagios、Zabbix、Prometheus、Logstash、Graylog、Prometheus、Grafana。

**可见性（Observability）**可以用来衡量 *可以从系统外部的输出来判断系统内部状态的程度*。Observability 有一些关键的数据类型（MELT）：Metrics、Events、Logs、Traces

#### Metrics

> 指标，提供活动或过程的信息的数字。比如 CPU 利用率、请求延迟 (ms)、错误率等

**Error**：错误，单位时间导致错误的请求比例

**Latency**：时延，完成一个单位任务的时间。需要统计所有请求的时延，不管请求成功与否。错误事务的时延在跟踪错误的时候很重要。时延可以用平均值或比率表示，比如 97% 的请求在 0.5s 内完成。

**Throughput**：单位时间完成的任务量。帮助量化系统的需求。

- Application：每秒的并行进程数
- Database：每秒执行的查询数
- Web Server：每秒处理的请求数

#### Events

> 事件是在某时刻发生的不彼此分离的行为，比如 Payment Microservice 在 2022/1/10 11:59pm 炸了

与 Metrics 不同，Events 发生的时间间隔是不确定的。

#### Logs

> 随着时间的流逝发生的各个事件的不可变的记录，在 databases、caches、load balancers 的排障中非常有用

Logs 可以有不同的格式：

- 随意的文本格式
- 一定的数据结构，比如 JSON 格式
- 二进制

可以用中心化的 Logs 记录项目中其他微服务的事件

##### ELK Stack

> ELK 指的是三个开源项目：Elasticsearch、Logstash 和 Kibana

Elasticsearch 是一个搜索和分析引擎

Logstash 是一个服务端的数据处理管道，他从多个数据来源获取数据，然后把它送给像 Elasticsearch 这样的 "stash"

Kibana 可以将 Elasticsearch 中的数据使用图标可视化

#### Traces

> 在分布式系统中，一个用户请求可能会经过好多微服务，这时候需要跟踪这个请求。

一个 Trace 是 Spans 构成的树，每一个的工作单元（独立的 services 或 components）就被称为一个 Span。 

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/MELT-Tracing.png)

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/MELT-Tracing-2.png)



### Communication and Collaboration

> 沟通与协作

可用工具：Rocket Chat、GitLab wiki、Redmine、Trello





