# NFR 需求计划 — 单元 2: unit-product（商品服务）

## 计划概览

基于功能设计产物和 INCEPTION 阶段已确定的技术栈，评估 Product Service 的非功能需求并确认技术选型细节。

## 已确定的技术栈（INCEPTION 阶段）

- Java 21 + Spring Boot 3.4.1 + MyBatis-Plus 3.5.7
- MySQL 8.4 + Druid 连接池 + Flyway 迁移
- Redis (Lettuce) 缓存
- jqwik PBT 框架
- JJWT 0.12.6（JWT）
- Logstash Logback Encoder + Micrometer Tracing

## 执行步骤

- [x] 步骤 1：确认性能和缓存需求
- [x] 步骤 2：确认测试策略细节
- [x] 步骤 3：生成 NFR 需求文档和技术栈决策文档

---

# NFR 需求问题

请回答以下问题，在 `[Answer]:` 后填写字母选项。

## Question 1
商品详情是否需要 Redis 缓存？（商品数据变更频率相对低，但读取频率高）

A) 是，商品详情使用 Redis 缓存，TTL 设为 10 分钟
B) 是，商品详情使用 Redis 缓存，TTL 设为 30 分钟
C) 否，当前阶段不做缓存，直接查数据库（后续优化时再加）
D) Other (please describe after [Answer]: tag below)

[Answer]:  B

## Question 2
分类树数据是否需要 Redis 缓存？（分类数据变更极少，但每次商品列表页都需要加载）

A) 是，分类树缓存到 Redis，TTL 设为 1 小时，分类变更时主动清除缓存
B) 否，分类数据量小，直接查数据库即可
C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 3
Product Service 的单元测试和集成测试使用什么数据库？

A) H2 内存数据库（测试速度快，但与 MySQL 有语法差异）
B) Testcontainers + MySQL 容器（与生产一致，但需要 Docker）
C) 直接连接本地 MySQL 测试库
D) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 4
API 响应时间的目标是什么？

A) 普通查询 < 200ms，库存扣减 < 500ms（含重试）
B) 普通查询 < 500ms，库存扣减 < 1000ms（含重试）
C) 当前阶段不设硬性指标，后续根据实际情况优化
D) Other (please describe after [Answer]: tag below)

[Answer]: B

