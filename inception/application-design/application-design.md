# 应用设计总览

本文档汇总应用设计的所有产物，详细内容请参阅各子文档。

## 设计概要

### 新增组件清单

| 服务 | 新增组件 | 类型 |
|------|---------|------|
| Auth Service | User、Authentication、TokenValidation | 领域组件 + 内部接口 |
| Product Service | Category | 领域组件（Product 已有，需补充方法） |
| Points Service | PointsAccount、PointsTransaction | 领域组件 |
| Order Service | Order、RedemptionOrchestrator、CompensationHandler | 领域组件 + 编排 + 补偿 |
| Frontend | AuthModule、ShopModule、EmployeeModule、AdminModule | 前端模块 |

### 新增数据库表

| 服务 | 表名 | 用途 |
|------|------|------|
| Auth | user | 用户信息 |
| Product | category | 商品分类（树形） |
| Points | points_account | 积分账户 |
| Points | points_transaction | 积分交易记录 |
| Order | redemption_order | 兑换订单 |

### 新增 API 端点（约 30+）

**Auth Service**：注册、登录、Token 验证、获取用户信息、用户列表
**Product Service**：商品详情、更新、删除、上下架、库存扣减/恢复、分类 CRUD、分类树
**Points Service**：积分余额、交易明细、扣减、退还、发放、批量发放
**Order Service**：兑换下单、员工订单列表、管理员订单列表、订单详情、取消订单、更新状态
**统计 API**：仪表盘统计、趋势数据、排行榜

### 跨服务通信

- **同步**：Order Service 通过 WebClient 调用 Points Service 和 Product Service
- **异步补偿**：失败补偿任务写入 Redis 队列，定时消费重试

### 关键设计决策

1. **兑换编排在 Order Service 应用层**：RedemptionApplicationService 负责跨服务编排，不在领域层
2. **补偿基于 Redis 队列**：同步调用失败时写入 Redis 补偿队列，异步重试
3. **分类使用 parentId 树形结构**：支持多级分类，前端递归渲染
4. **商品 category 字段改为 categoryId**：关联分类表，替代原有字符串

## 详细文档索引

- [组件定义](components.md) — 各服务组件职责和领域模型
- [组件方法签名](component-methods.md) — 领域服务、应用服务、前端 API 方法签名
- [服务编排](services.md) — 服务层级、编排流程、补偿策略
- [组件依赖关系](component-dependency.md) — 服务间依赖、内部分层依赖、数据流
