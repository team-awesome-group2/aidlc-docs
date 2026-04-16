# 技术栈决策 — unit-product

## 已确定技术栈（继承自项目级决策）

| 类别 | 技术 | 版本 | 决策来源 |
|------|------|------|---------|
| 语言 | Java | 21 | 项目既有 |
| 框架 | Spring Boot | 3.4.1 | 项目既有 |
| ORM | MyBatis-Plus | 3.5.7 | 项目既有 |
| 数据库 | MySQL | 8.4 | 项目既有 |
| 连接池 | Druid | — | 项目既有 |
| 迁移 | Flyway | — | 项目既有 |
| 缓存 | Spring Data Redis (Lettuce) | — | 项目既有 |
| JWT | JJWT | 0.12.6 | 项目既有 |
| API 文档 | SpringDoc OpenAPI | — | 项目既有 |
| 日志 | Logstash Logback Encoder | 7.4 | 项目既有 |
| 追踪 | Micrometer Tracing | 1.3.5 | 项目既有 |

## 本单元新增技术决策

| 决策项 | 选择 | 理由 |
|--------|------|------|
| PBT 框架 | jqwik | JUnit 5 集成、支持自定义生成器和 shrinking、INCEPTION 阶段已确定 |
| 商品详情缓存 | Redis Cache-Aside, TTL 30min | 读多写少场景，30 分钟平衡数据新鲜度和性能 |
| 分类树缓存 | Redis Cache-Aside, TTL 1h + 主动清除 | 分类变更极少，1 小时 TTL + 变更时主动清除保证一致性 |
| 测试数据库 | 本地 MySQL（application-test.yml） | 与生产环境一致，避免 H2 语法差异，无需 Docker 依赖 |
| 库存扣减重试 | 应用层重试 3 次 | 乐观锁冲突窗口短，立即重试成功率高 |

## 不新增依赖

本单元不需要引入任何新的 Maven 依赖。所有需要的库（Spring Boot、MyBatis-Plus、Redis、jqwik 等）已在项目 pom.xml 中声明。

**注意**：根据项目规范，pom.xml 未经确认不得修改。如果 jqwik 尚未在 pom.xml 中声明，需要在代码生成阶段确认并请求添加。
