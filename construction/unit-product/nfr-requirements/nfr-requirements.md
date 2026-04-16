# NFR 需求 — unit-product

## 1. 性能需求

### NFR-PERF-01: API 响应时间

| 操作类型 | 目标响应时间 | 说明 |
|---------|------------|------|
| 普通查询（列表、详情、分类树） | < 500ms | 含缓存命中场景 |
| 库存扣减（含乐观锁重试） | < 1000ms | 最多 3 次重试 |
| 库存恢复 | < 500ms | 无重试 |
| 商品创建/更新/删除 | < 500ms | 含缓存清除 |

### NFR-PERF-02: 分页限制

- 默认每页 20 条，最大 100 条
- 禁止无分页的全量查询（分类树除外，数据量可控）

---

## 2. 缓存需求

### NFR-CACHE-01: 商品详情缓存

- 缓存位置：Redis
- 缓存 Key：`product:detail:{productId}`
- TTL：30 分钟
- 缓存策略：Cache-Aside（旁路缓存）
  - 读取：先查 Redis，miss 则查 DB 并写入 Redis
  - 写入/更新/删除/状态变更：先更新 DB，再删除 Redis 缓存（下次读取时重建）
- 缓存值：ProductEntity 序列化为 JSON
- 缓存穿透防护：查询不存在的商品时缓存空值，TTL 5 分钟

### NFR-CACHE-02: 分类树缓存

- 缓存位置：Redis
- 缓存 Key：`category:tree`
- TTL：1 小时
- 缓存策略：
  - 读取：先查 Redis，miss 则查 DB 组装树并写入 Redis
  - 分类变更（创建/更新/删除）：主动清除缓存 Key
- 缓存值：完整分类树 JSON

### NFR-CACHE-03: 缓存端口设计

- 在 `domain/cache-api` 中定义缓存端口接口
- 在 `infrastructure/cache/redis-impl` 中实现
- 领域服务通过端口接口操作缓存，不直接依赖 Redis

---

## 3. 并发控制

### NFR-CONC-01: 库存扣减乐观锁

- 使用 MyBatis-Plus `@Version` 乐观锁
- SQL 层面：`WHERE id = ? AND version = ? AND stock >= ?`
- 冲突重试：最多 3 次，每次重新读取最新数据
- 重试间无延迟（立即重试，因为冲突窗口极短）

### NFR-CONC-02: 商品更新乐观锁

- 使用 MyBatis-Plus `@Version` 乐观锁
- 冲突时不重试，直接返回错误（由用户重新操作）

---

## 4. 安全需求

### NFR-SEC-01: 端点权限分层

| 路径前缀 | 权限要求 | 说明 |
|---------|---------|------|
| /api/v1/public/ | 已认证用户 | 员工和管理员均可访问 |
| /api/v1/admin/ | 管理员角色 | 仅管理员可访问 |
| /api/v1/internal/ | 无用户权限校验 | 仅服务间调用，Gateway 不对外暴露 |

### NFR-SEC-02: 输入校验

- 所有 Request DTO 使用 Jakarta Validation 注解
- 字符串字段设置 @Size 最大长度
- 数值字段设置 @Min/@Max 范围
- 必填字段使用 @NotNull/@NotBlank

### NFR-SEC-03: SQL 注入防护

- 所有数据库操作使用 MyBatis-Plus 参数化查询
- 模糊搜索使用 `CONCAT('%', #{name}, '%')`，不拼接 SQL

---

## 5. 可观测性

### NFR-OBS-01: 结构化日志

- 使用已有的 Logstash Logback Encoder
- 关键操作记录日志：库存扣减/恢复、商品状态变更、分类删除
- 日志包含：timestamp、requestId、operatorId、操作类型、关键参数
- 不记录敏感信息

### NFR-OBS-02: 指标暴露

- 使用已有的 Actuator + Prometheus 端点
- 无需额外自定义指标（当前阶段）

---

## 6. 测试需求

### NFR-TEST-01: 单元测试

- 覆盖领域服务核心业务逻辑
- Mock 仓储层和缓存层
- 框架：JUnit 5 + Mockito

### NFR-TEST-02: 集成测试

- 覆盖关键 API 端点（Controller → DB 全链路）
- 数据库：直连本地 MySQL 测试库（application-test.yml 配置）
- 框架：Spring Boot Test + @SpringBootTest

### NFR-TEST-03: 属性测试（PBT）

- 框架：jqwik（已在 INCEPTION 阶段确定）
- 覆盖功能设计中识别的 9 个可测试属性
- 自定义生成器：ProductEntity 生成器、CategoryEntity 生成器
- 与 JUnit 5 集成运行

---

## 7. 数据完整性

### NFR-DATA-01: 逻辑删除

- 所有表使用逻辑删除（deleted 字段）
- MyBatis-Plus @TableLogic 自动过滤

### NFR-DATA-02: 审计字段

- 所有表包含 created_at、updated_at、created_by、updated_by
- 通过 MyBatis-Plus MetaObjectHandler 自动填充
