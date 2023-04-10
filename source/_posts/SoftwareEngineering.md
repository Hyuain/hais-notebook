---
title: Software Engineering
date: 2023-04-10 14:34:12
categories:
  - [计算机]
---

.

<!-- more -->

# DevOps

> DevOps = Development + Operation = New Mindset + New Tools + New Skills

DevOps 主要是通过一系列的原则、工具、方法来提高开发与运维的写作，通过快速发布新功能、修复和优化来缩短开发周期。

敏捷开发加速了业务需求 -> 开发，DevOps 则加速了开发 -> 运维。

**DevOps 运用到的技术与工具：**

- **Continuous Integration**：持续集成
  - **Source Code Management**：源代码管理，Git、SVN、CVS、ClearCase
  - **Build Process**：构建，Maven、Gradle、Rake、JUnit、Sonar
  - **Continuous Integration**：持续集成，Jenkins、GitLab CI、Travis、Nexus
  - **Test Automation**：自动化测试，包括单元测试、服务测试、UI 测试等
- **Continuous Delivery**：持续交付
  - **Deployment Automation**：自动化部署，Chef、Puppet、Docker、Heroku、Open Stack、AWS Cloud Formation、Rancher、Flyway
- **Microservices**：微服务
- **Infrastructure as Code**：基础设置及代码
  - 使用机器可读的定义文件来管理开发、测试和生产环境
- **Monitoring and Logging**：监测与日志
  - 让开发者查看生产环境指标，Nagios、Zabbix、Prometheus、Logstash、Graylog
- **Communication and Collaboration**：沟通与协作
  - Knowledge Sharing：知识库，Rocket Chat、GitLab wiki、Redmine、Trello

**DevOps 表现指标：**

- **Delivery Lead Time**：交付前时间，包括代码实现、测试、提交更改到生产系统，越短越好
- **Deployment Frequency**：部署频率，越高越好
- **Change Fail Percentage**：变更导致需要回滚或紧急修复的比率，越低越好
- **Mean Time to Restore (MTTR)**：从突发事件中恢复的平均时间，越短越好

## CI & CD

- **Continuous Integration**
  - 每个开发人员开发开发自己负责的部分
  - 需要持续地将自己的代码与其他的人的代码集成起来
  - 需要对集成后的代码进行单元测试、集成测试、回归测试
- **Continuous Delivery**
  - 持续交付并不是说尽可能快地部署所有变更，而是让所有变更在任何时刻都被保证是可部署的
  - 指的是部署在任何时间、任何平台部署任意版本的能力和基础设施
- **From CI to CD**
  - 集成所有该项目相关的工作，以及所有构成该服务、应用或系统的代码
  - 验证构建完成无错误，将应用交付给 QA（代码从开发环境交付到测试环境）
  - 在模拟环境下进行冒烟测试、接受性测试、系统稳定性测试，提供给运维一些必要的脚本和测试数据来进行接受性测试和性能测试
  - 在类生产环境中进行测试

