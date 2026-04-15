# 工作单元定义

## 拆分策略

**拆分依据**：按现有微服务边界拆分，每个服务为一个独立工作单元。前端作为最后一个单元，依赖所有后端 API 就绪。

**执行顺序**：基于服务间依赖关系确定，无依赖的服务优先，有依赖的服务后置。

---

## 单元 1：Auth Service（认证服务）

| 属性 | 值 |
|------|------|
| **单元名称** | unit-auth |
| **对应仓库** | awsome-shop-auth-service |
| **优先级** | 最高（其他服务依赖 Token 验证） |
| **依赖** | 无外部服务依赖 |
| **范围** | 用户注册/登录/Token验证/登录限流/用户查询 |
| **新增组件** | User、Authentication、TokenValidation |
| **新增数据库表** | user |
| **关联故事** | US-AUTH-01~07（7 个） |

**交付物**：

- 用户领域模型 + 领域服务 + 仓储实现
- 认证领域服务（登录/JWT/限流）
- Token 验证内部接口
- 用户管理 API
- Flyway 迁移脚本
- 单元测试 + 集成测试 + PBT

---

## 单元 2：Product Service（商品服务）

| 属性 | 值 |
|------|------|
| **单元名称** | unit-product |
| **对应仓库** | awsome-shop-product-service |
| **优先级** | 高（与 Points 并行，Order 依赖它） |
| **依赖** | 无外部服务依赖 |
| **范围** | 商品 CRUD 补全/上下架/库存管理/分类管理 |
| **新增组件** | Category（Product 已有，补充方法） |
| **新增数据库表** | category |
| **关联故事** | US-PROD-01~09（9 个） |

**交付物**：

- 分类领域模型 + 领域服务 + 仓储实现
- 商品服务补充（详情/更新/删除/上下架/库存扣减恢复）
- 分类管理 API
- 商品内部接口（供 Order Service 调用）
- Flyway 迁移脚本
- 单元测试 + 集成测试 + PBT

---

## 单元 3：Points Service（积分服务）

| 属性 | 值 |
|------|------|
| **单元名称** | unit-points |
| **对应仓库** | awsome-shop-points-service |
| **优先级** | 高（与 Product 并行，Order 依赖它） |
| **依赖** | 无外部服务依赖 |
| **范围** | 积分账户/交易记录/余额查询/扣减/退还/发放 |
| **新增组件** | PointsAccount、PointsTransaction |
| **新增数据库表** | points_account、points_transaction |
| **关联故事** | US-PTS-01~06（6 个） |

**交付物**：

- 积分账户领域模型 + 领域服务 + 仓储实现
- 积分交易领域模型 + 领域服务 + 仓储实现
- 积分查询/发放 API
- 积分内部接口（供 Order Service 调用：扣减/退还）
- Flyway 迁移脚本
- 单元测试 + 集成测试 + PBT

---

## 单元 4：Order Service（订单服务）

| 属性 | 值 |
|------|------|
| **单元名称** | unit-order |
| **对应仓库** | awsome-shop-order-service |
| **优先级** | 中（依赖 Product + Points 就绪） |
| **依赖** | Product Service（库存）、Points Service（积分） |
| **范围** | 兑换下单/订单查询/订单取消/状态管理/跨服务编排/补偿 |
| **新增组件** | Order、RedemptionOrchestrator、CompensationHandler |
| **新增数据库表** | redemption_order |
| **关联故事** | US-ORD-01~06（6 个） |

**交付物**：

- 订单领域模型 + 领域服务 + 仓储实现
- 兑换编排应用服务（跨服务调用）
- 跨服务客户端（PointsServiceClient、ProductServiceClient）
- Redis 补偿队列 + CompensationHandler
- 订单管理 API
- Flyway 迁移脚本
- 单元测试 + 集成测试 + PBT

---

## 单元 5：Gateway Service（网关服务）

| 属性 | 值 |
|------|------|
| **单元名称** | unit-gateway |
| **对应仓库** | awsome-shop-gateway-service |
| **优先级** | 中低（Auth Token 验证端点就绪后即可验证） |
| **依赖** | Auth Service（Token 验证端点） |
| **范围** | CORS 配置优化 |
| **新增组件** | 无（仅配置调整） |
| **关联故事** | 无独立故事（FR-05.1） |

**交付物**：

- CORS 配置按环境区分（local 宽松 / prod 收紧）
- 验证与 Auth Service Token 验证端点的集成

---

## 单元 6：Frontend（前端）

| 属性 | 值 |
|------|------|
| **单元名称** | unit-frontend |
| **对应仓库** | awsome-shop-frontend |
| **优先级** | 最后（依赖所有后端 API 就绪） |
| **依赖** | Auth + Product + Points + Order 全部后端服务 |
| **范围** | 登录/注册对接、员工端 5 个页面、管理端 5 个页面、仪表盘 |
| **新增组件** | AuthModule、ShopModule、EmployeeModule、AdminModule |
| **关联故事** | US-FE-01~13（13 个） |

**交付物**：

- API 服务层（authApi/productApi/pointsApi/orderApi/adminApi）
- 登录/注册页面对接
- 员工端：商城首页（真实数据）、商品详情页、兑换流程、兑换记录页、积分明细页
- 管理端：商品管理、分类管理、订单管理、积分管理、用户管理
- 管理仪表盘（真实数据 + 趋势图表 + 排行榜）
- 单元测试 + PBT（fast-check）
