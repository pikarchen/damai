# 🎫 StarTicket-Cloud 分布式高并发票务系统

[![Spring Boot](https://img.shields.io/badge/SpringBoot-2.x%20/%203.x-green.svg)](https://spring.io/projects/spring-boot)
[![Spring Cloud Alibaba](https://img.shields.io/badge/SpringCloudAlibaba-Latest-blue.svg)](https://github.com/alibaba/spring-cloud-alibaba)
[![Redis](https://img.shields.io/badge/Redis-Latest-red.svg)](https://redis.io/)
[![Kafka](https://img.shields.io/badge/Kafka-Latest-black.svg)](https://kafka.apache.org/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

## 📖 项目简介

StarTicket-Cloud 是一款基于微服务架构设计的在线高并发票务抢购系统。系统涵盖了演唱会、话剧、体育比赛等节目的在线浏览、座位选择、高并发抢票、订单支付及状态流转等完整闭环业务。

本项目不仅实现了基础的票务流转，更针对“热门节目秒杀”、“海量用户瞬时涌入”等真实业务场景，从链路限流、多级缓存、分布式事务、异步削峰等多个维度进行了深度架构优化，是一套具备高吞吐量和高可用性的企业级实践方案。

## 🏗️ 核心系统架构

系统采用 `Spring Cloud + Spring Cloud Alibaba` 微服务生态，并融合多种中间件以应对复杂的并发场景。

*(💡 建议：在此处放上你下载到本地的架构图，例如：`![系统架构图](./docs/images/architecture.png)`)*

### 核心技术栈
* **微服务治理**：Nacos (注册/配置中心)、Gateway (网关)、Sentinel (熔断限流)
* **核心业务支撑**：Spring Boot, MyBatis-Plus, ShardingSphere (分库分表)
* **高并发&异步中间件**：Redis, Redisson, Kafka
* **监控与日志**：ELK (Elasticsearch, Logstash, Kibana), Spring Boot Admin
* **安全与认证**：JWT, AJ-Captcha (图形验证码)

## ✨ 核心特性与技术方案

本项目针对高并发场景下的常见痛点，落地了以下技术解决方案：

### 1. 极致的缓存架构 (应对读多写少的高并发查询)
* **防穿透与防击穿**：结合**布隆过滤器 (Bloom Filter)**与**空值缓存**策略，将恶意请求拦截在系统最外层；针对热点数据失效问题，采用**双重锁检测 (DCL) + 逻辑过期异步重建**方案，彻底消除缓存击穿风险。
* **多级缓存体系**：在首页列表、节目详情等超高频查询接口，采用 `本地缓存 (Caffeine/Guava) + 分布式缓存 (Redis)` 的多级缓存架构，极大降低网络 IO 与 Redis 集群压力。

### 2. 高可靠的库存扣减与抢购链路 (并发写控制)
* **全链路流控**：抢票瞬时采用“令牌桶限流 + 验证码错峰 + 网关全局限流”，保障核心服务不被洪峰压垮。
* **分布式锁深度优化**：摒弃粗粒度的锁实现，针对不同购票场景精细化使用 Redisson 的读写锁、公平锁；结合业务需求定制化处理锁超时与事务边界问题。
* **一致性闭环**：通过 `Redis + Lua 脚本` 实现原子性的库存预扣减，结合 `Kafka 异步消息队列` 与本地消息表（Outbox Pattern），保障缓存与数据库间的高最终一致性。

### 3. 高可用设计与故障降级
* **中间件容灾处理**：针对 Redis 或 Kafka 可能出现的宕机情况，设计了完善的**异常捕获、重试退避与对账日志补偿机制**，确保在极端情况下核心数据不丢失、订单状态可回滚可恢复。
* **消息可靠性保障**：Kafka 消费端实现严格的幂等性校验，并引入死信队列 (DLQ) 处理超时或失败的死信消息。

### 4. 海量数据存储与扩展
* **分库分表**：面对庞大的订单和用户数据，基于 `ShardingSphere` 设计了合理的路由拆分键（如按用户 ID 或订单 ID Hash），在防止读扩散的同时，保障了单表数据量的可控性。
* **全局唯一标识**：采用定制化的高性能雪花算法（Snowflake）/ 发号器，解决分布式环境下的订单号生成与分表 ID 冲突问题。

## ⚙️ 业务模块与界面展示

系统尽可能还原了真实商业票务平台的交互体验，主要包含以下核心模块：
1. **C端用户门户**：主页推荐、多维度节目检索、节目详情查阅。
2. **交易核心**：选座锁座、订单生成、支付状态轮询、订单延迟关闭（基于分布式延迟队列）。
3. **用户中心**：高并发注册防刷、个人订单管理与状态追踪。

*(💡 建议：在此放 3-4 张最核心的系统截图，横向排布或使用 HTML 缩小尺寸，不要占用过多篇幅。同样需要保存为本地图片引用。)*

## 🚀 快速本地启动

### 1. 环境准备
请确保本地已安装以下基础环境：
* JDK 17+ (或 8+ 根据你的实际版本调整)
* MySQL 8.0+
* Redis 7.0+
* Kafka & Zookeeper
* Nacos (建议单机启动用于开发测试)

### 2. 数据库初始化
1. 创建数据库 `star_ticket_core`。
2. 导入项目 `sql` 目录下的 `init.sql` 与分库分表需要的结构文件。

### 3. 修改配置
在对应微服务的 `application.yml` 或 Nacos 配置中心中，修改数据库连接、Redis 密码及 Kafka 地址为您本地的配置。

### 4. 启动服务
建议按照以下顺序启动微服务模块：
1. `GatewayApplication` (网关)
2. `UserApplication` (用户服务)
3. `ProgramApplication` (节目/票务服务)
4. `OrderApplication` (订单服务)

## 🤝 贡献与交流

欢迎提交 Issue 探讨架构设计上的优化点或报告 Bug。如果你对高并发架构有更好的想法，欢迎提交 Pull Request 一起完善该系统。

---
*本项目作为个人对高并发、分布式架构落地的实践总结。*
