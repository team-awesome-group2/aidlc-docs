# Requirements Document — AWSome Shop 二次开发

## Intent Analysis Summary

- **User Request**: 基于现有 AWSome Shop 电商平台进行二次开发，完善现有功能 + 新增 Phase 1（核心闭环）+ Phase 2（管理能力）
- **Request Type**: Enhancement + New Feature
- **Scope Estimate**: System-wide（涉及全部 6 个服务）
- **Complexity Estimate**: Complex（多服务协调、跨服务事务、前后端全栈）
- **Requirements Depth**: Comprehensive

---

## 1. Functional Requirements

### FR-01: Auth Service — 用户认证与授权

#### FR-01.1: 用户注册

- 员工可通过公开接口自行注册账号
- 管理员可通过后台创建用户账号
- 注册信息：用户名、密码、显示名称、角色（employee/admin）
- 用户名唯一性校验
- 密码使用 BCrypt 自适应哈希存储

#### FR-01.2: 用户登录

- 用户名 + 密码登录
- 登录成功返回 JWT Token + 用户信息
- Token 有效期 2 小时，过期后需重新登录（不做无感刷新）

#### FR-01.3: Token 验证（内部接口）

- 实现 `POST /api/v1/internal/auth/validate` 端点
- 接收 JWT Token，验证签名和有效期
- 返回 operatorId（用户 ID）供 Gateway 使用

#### FR-01.4: 登录安全

- 登录失败限流：5 次失败后锁定 30 分钟（配置已存在）
- 实现登录失败计数逻辑（基于 Redis）

#### FR-01.5: 用户信息查询

- 获取当前用户信息接口（基于 Token 中的 operatorId）
- 管理员查询用户列表接口（分页）

#### FR-01.6: 数据库

- 用户表：id, username, password_hash, display_name, role, status, points_balance(可选), created_at, updated_at, deleted, version
- Flyway 迁移脚本

### FR-02: Product Service — 商品服务完善

#### FR-02.1: 完善现有 API

- 获取单个商品详情 API
- 更新商品 API（含验证）
- 删除商品 API（逻辑删除）

#### FR-02.2: 商品状态管理

- 上架/下架操作 API
- 仅上架商品对员工可见

#### FR-02.3: 库存管理

- 库存扣减接口（供 Order Service 调用）
- 库存恢复接口（订单取消时调用）
- 库存不足时拒绝兑换

#### FR-02.4: 商品分类管理

- 支持多级分类（树形结构）
- 分类 CRUD API
- 分类表：id, name, parent_id, sort_order, status, created_at, updated_at, deleted
- 商品关联分类 ID（替代当前的字符串 category）

### FR-03: Points Service — 积分服务（全新实现）

#### FR-03.1: 积分账户

- 每个用户一个积分账户
- 积分账户表：id, user_id, balance, total_earned, total_spent, created_at, updated_at, version

#### FR-03.2: 积分交易记录

- 记录每笔积分变动（收入/支出）
- 交易记录表：id, user_id, type(EARN/SPEND/REFUND), amount, balance_after, reference_type, reference_id, description, created_at
- reference_type/reference_id 关联业务来源（如 ORDER/ADMIN_GRANT）

#### FR-03.3: 积分查询

- 查询当前积分余额 API
- 查询积分交易明细 API（分页，支持按类型筛选）

#### FR-03.4: 积分扣减

- 兑换时扣减积分接口（供 Order Service 调用）
- 余额不足时拒绝扣减
- 使用乐观锁防止并发超扣

#### FR-03.5: 积分退还

- 订单取消时退还积分接口
- 生成 REFUND 类型交易记录

#### FR-03.6: 积分发放

- 管理员手动发放积分 API（单个用户）
- 管理员批量发放积分 API（多个用户）
- 生成 EARN 类型交易记录

### FR-04: Order Service — 订单服务（全新实现）

#### FR-04.1: 创建兑换订单

- 员工提交兑换请求（商品 ID、数量）
- 业务流程：
  1. 校验商品状态（已上架）
  2. 校验库存充足
  3. 校验积分余额充足
  4. 扣减积分（调用 Points Service）
  5. 扣减库存（调用 Product Service）
  6. 创建订单记录
- 跨服务一致性：本地事务 + Redis 队列实现最终一致性
- 任一步骤失败时执行补偿（退还积分、恢复库存）

#### FR-04.2: 订单状态流转

- 状态：PENDING → PROCESSING → COMPLETED / CANCELLED
- 管理员可更新订单状态

#### FR-04.3: 订单查询

- 员工查询自己的订单列表（分页）
- 管理员查询所有订单列表（分页，支持按状态筛选）
- 查询订单详情

#### FR-04.4: 订单取消

- 允许取消，立即退还积分和恢复库存
- 仅 PENDING 和 PROCESSING 状态可取消
- 取消后状态变为 CANCELLED

#### FR-04.5: 数据库

- 订单表：id, order_no, user_id, product_id, product_name, quantity, points_amount, status, created_at, updated_at, deleted, version
- 订单号生成规则（如：ORD + 时间戳 + 随机数）

### FR-05: Gateway Service — 小优化

#### FR-05.1: CORS 收紧

- 生产环境限制允许的来源域名（本地开发保持宽松）

### FR-06: Frontend — 前端对接与页面补全

#### FR-06.1: 对接后端认证

- 替换 Mock 登录，调用后端 Auth API
- Token 存储到 localStorage
- 401 响应自动跳转登录页（已实现）

#### FR-06.2: 员工端页面

- 商城首页：对接商品列表 API，支持分类筛选和搜索
- 商品详情页：展示商品详情 + 兑换按钮
- 兑换流程：确认弹窗 → 调用兑换 API → 结果反馈
- 兑换记录页（/orders）：对接订单列表 API
- 积分明细页（/points）：对接积分交易记录 API

#### FR-06.3: 管理端页面

- 商品管理（/admin/products）：商品列表 + 新增/编辑/删除/上下架
- 分类管理（/admin/categories）：树形分类管理
- 订单管理（/admin/orders）：订单列表 + 状态更新
- 积分管理（/admin/points）：积分发放（单个/批量）
- 用户管理（/admin/users）：用户列表 + 创建用户

#### FR-06.4: 管理仪表盘（真实数据）

- 基础统计：商品总数、用户总数、本月兑换量、积分流通量
- 趋势图表：近 7 天/30 天兑换趋势、积分消耗趋势
- 排行榜：热门商品 Top10、活跃用户 Top10

---

## 2. Non-Functional Requirements

### NFR-01: API 风格

- 保持现有 POST-only 风格，所有新增 API 也使用 POST + JSON Body

### NFR-02: 跨服务通信

- Order Service 调用 Points/Product Service 使用同步 HTTP（WebClient）
- 补偿逻辑使用本地事务 + Redis 队列实现最终一致性
- 不使用 AWS SQS（使用 Redis 作为本地队列方案）

### NFR-03: 测试策略

- 核心业务逻辑需要单元测试
- 关键 API 需要集成测试
- Java 服务使用 jqwik 框架进行属性测试（PBT）
- 前端使用 fast-check 进行属性测试（PBT）

### NFR-04: 安全

- 启用 Security Baseline 扩展（SECURITY-01 至 SECURITY-15）
- JWT 认证 + BCrypt 密码哈希 + AES-256 加密
- 登录限流防暴力破解
- 输入验证（所有 API 参数）
- 全局异常处理（不暴露内部信息）

### NFR-05: 性能

- 积分扣减使用乐观锁防止并发超扣
- 商品列表支持分页（默认 20 条，最大 100 条）
- Redis 缓存热点数据（商品详情、积分余额）

### NFR-06: 可观测性

- 结构化日志（已有 Logstash Logback Encoder）
- 请求追踪（已有 Micrometer Tracing）
- Prometheus 指标暴露（已有 /actuator/prometheus）

### NFR-07: 数据库

- 每个服务独立数据库
- Flyway 管理迁移
- 逻辑删除 + 乐观锁（已有模式）

---

## 3. Technical Decisions Summary

| Decision | Choice | Rationale |
|----------|--------|-----------|
| 用户注册 | 员工自注册 + 管理员创建 | 灵活性 |
| 积分发放 | 仅管理员手动发放 | MVP 阶段简化 |
| 跨服务一致性 | 本地事务 + Redis 队列 | 利用现有 Redis，避免 SQS 依赖 |
| 商品分类 | 多级分类（树形） | 支持更细粒度的分类管理 |
| 仪表盘 | 基础统计 + 趋势 + 排行榜 | 完整的运营数据视图 |
| Token 刷新 | 不做无感刷新 | 简化实现，2 小时有效期足够 |
| API 风格 | POST-only | 保持与现有代码一致 |
| 订单取消 | 允许取消，立即退还 | 用户友好 |
| 测试 | 单元测试 + 集成测试 + PBT | 全面质量保障 |
| Security | 启用全部安全规则 | 生产级应用标准 |
| PBT | 启用全部 PBT 规则 | 全面属性测试覆盖 |
