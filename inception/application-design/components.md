# 组件定义

## 1. Auth Service 组件

### 1.1 User（用户组件）

- **职责**：用户注册、用户信息管理、用户查询
- **领域模型**：UserEntity（id, username, passwordHash, displayName, role, status, createdAt, updatedAt）
- **所属层**：domain-model / domain-api / domain-impl / repository-api

### 1.2 Authentication（认证组件）

- **职责**：用户登录、JWT Token 生成与验证、登录限流
- **领域模型**：无独立实体，依赖 UserEntity + Redis 存储登录状态
- **所属层**：domain-api / domain-impl / security-api（JWT Port）/ cache-api（Redis Port）

### 1.3 TokenValidation（Token 验证组件 — 内部接口）

- **职责**：为 Gateway 提供 Token 验证端点
- **所属层**：interface-http（内部控制器）

---

## 2. Product Service 组件

### 2.1 Product（商品组件 — 已有，需完善）

- **职责**：商品 CRUD、上下架管理、库存扣减/恢复
- **领域模型**：ProductEntity（已有，需新增 categoryId 字段替代 category 字符串）
- **所属层**：已有全部层，需补充方法

### 2.2 Category（分类组件 — 新增）

- **职责**：多级分类树管理（CRUD）
- **领域模型**：CategoryEntity（id, name, parentId, sortOrder, status, createdAt, updatedAt）
- **所属层**：domain-model / domain-api / domain-impl / repository-api / application-api / application-impl / interface-http

---

## 3. Points Service 组件

### 3.1 PointsAccount（积分账户组件 — 新增）

- **职责**：积分余额管理、积分扣减、积分退还、积分发放
- **领域模型**：PointsAccountEntity（id, userId, balance, totalEarned, totalSpent, createdAt, updatedAt, version）
- **所属层**：domain-model / domain-api / domain-impl / repository-api

### 3.2 PointsTransaction（积分交易组件 — 新增）

- **职责**：积分交易记录管理、交易明细查询
- **领域模型**：PointsTransactionEntity（id, userId, type, amount, balanceAfter, referenceType, referenceId, description, createdAt）
- **所属层**：domain-model / domain-api / domain-impl / repository-api

---

## 4. Order Service 组件

### 4.1 Order（订单组件 — 新增）

- **职责**：兑换下单、订单查询、订单取消、订单状态流转
- **领域模型**：OrderEntity（id, orderNo, userId, productId, productName, productImage, quantity, pointsAmount, status, createdAt, updatedAt, version）
- **所属层**：domain-model / domain-api / domain-impl / repository-api

### 4.2 RedemptionOrchestrator（兑换编排组件 — 新增）

- **职责**：编排跨服务兑换流程（校验 → 扣积分 → 扣库存 → 创建订单 → 失败补偿）
- **所属层**：application-impl（应用服务层编排，不在领域层）
- **依赖**：Points Service（HTTP 调用）、Product Service（HTTP 调用）

### 4.3 CompensationHandler（补偿处理组件 — 新增）

- **职责**：兑换失败时的补偿逻辑（退还积分、恢复库存），基于 Redis 队列
- **所属层**：infrastructure（Redis 队列实现）

---

## 5. Gateway Service 组件（已有，小优化）

### 5.1 AuthenticationFilter（已有）

- **职责**：JWT Token 验证过滤器
- **优化**：CORS 配置按环境区分

---

## 6. Frontend 组件

### 6.1 AuthModule（认证模块）

- **职责**：登录/注册页面、Token 管理、对接后端 Auth API
- **文件**：pages/Login、pages/Register、store/useAuthStore、services/authApi

### 6.2 ShopModule（员工商城模块）

- **职责**：商品浏览、商品详情、兑换流程
- **文件**：pages/ShopHome、pages/ProductDetail、services/productApi

### 6.3 EmployeeModule（员工个人模块）

- **职责**：兑换记录、积分明细
- **文件**：pages/Orders、pages/Points、services/orderApi、services/pointsApi

### 6.4 AdminModule（管理后台模块）

- **职责**：商品管理、分类管理、订单管理、积分管理、用户管理、仪表盘
- **文件**：pages/admin/*、services/adminApi
