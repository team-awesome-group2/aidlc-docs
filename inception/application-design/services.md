# 服务定义与编排

## 服务层级说明

按 DDD 六边形架构，服务分为三层：

- **领域服务（Domain Service）**：封装核心业务逻辑，不依赖基础设施
- **应用服务（Application Service）**：编排领域服务，处理 DTO 转换，协调跨组件操作
- **接口层（Interface）**：HTTP 控制器，调用应用服务

---

## 1. Auth Service 服务编排

### 应用服务：UserApplicationService

- **职责**：用户注册、用户查询的 DTO 转换和编排
- **编排**：调用 UserDomainService

### 应用服务：AuthApplicationService

- **职责**：登录流程编排
- **编排流程**：
  1. 调用 AuthenticationDomainService.checkLoginAttempts() 检查是否锁定
  2. 调用 UserDomainService.getByUsername() 获取用户
  3. 验证密码
  4. 失败 → 调用 recordFailedAttempt()
  5. 成功 → 调用 resetFailedAttempts() + 生成 JWT Token

### 应用服务：TokenValidationApplicationService

- **职责**：为 Gateway 提供 Token 验证
- **编排**：调用 AuthenticationDomainService.validateToken()

---

## 2. Product Service 服务编排

### 应用服务：ProductApplicationService（已有，需补充）

- **职责**：商品 CRUD 编排
- **新增编排**：
  - update：DTO → Entity 转换 → 调用 ProductDomainService.update()
  - delete：调用 ProductDomainService.delete()
  - updateStatus：调用 ProductDomainService.updateStatus()
  - deductStock / restoreStock：调用对应领域服务方法

### 应用服务：CategoryApplicationService（新增）

- **职责**：分类管理编排
- **编排**：调用 CategoryDomainService 的 CRUD 方法

---

## 3. Points Service 服务编排

### 应用服务：PointsApplicationService（新增）

- **职责**：积分查询、发放、扣减、退还的编排
- **编排**：
  - getBalance：调用 PointsAccountDomainService.getByUserId()
  - getTransactions：调用 PointsTransactionDomainService.page()
  - deduct：调用 PointsAccountDomainService.deduct()（事务内同时写交易记录）
  - refund：调用 PointsAccountDomainService.refund()
  - grant / batchGrant：调用 PointsAccountDomainService.grant()

---

## 4. Order Service 服务编排

### 应用服务：OrderApplicationService（新增）

- **职责**：订单查询、状态管理
- **编排**：调用 OrderDomainService 的查询和状态更新方法

### 应用服务：RedemptionApplicationService（新增 — 核心编排）

- **职责**：兑换下单的跨服务编排
- **编排流程**：

  ```
  redeem(userId, productId, quantity):
    1. 调用 ProductServiceClient.getProduct(productId) → 校验商品状态和库存
    2. 计算积分金额 = product.pointsPrice * quantity
    3. 调用 PointsServiceClient.deductPoints(userId, amount, orderNo) → 扣减积分
    4. 调用 ProductServiceClient.deductStock(productId, quantity) → 扣减库存
       - 失败 → 调用 PointsServiceClient.refundPoints() 补偿
    5. 调用 OrderDomainService.create() → 创建订单
       - 失败 → 调用补偿（退还积分 + 恢复库存）
    6. 返回订单信息
  ```

- **补偿策略**：

  ```
  compensate(context):
    - 如果积分已扣减 → 调用 refundPoints()
    - 如果库存已扣减 → 调用 restoreStock()
    - 补偿失败 → 写入 Redis 补偿队列，异步重试
  ```

### 应用服务：CompensationService（新增）

- **职责**：处理 Redis 补偿队列中的失败补偿任务
- **编排**：定时从 Redis 队列读取补偿任务，重试执行

---

## 5. 跨服务通信模式

### 同步调用（WebClient）

```
Order Service ──HTTP POST──→ Points Service（扣减/退还积分）
Order Service ──HTTP POST──→ Product Service（查询商品/扣减/恢复库存）
```

### 异步补偿（Redis 队列）

```
Order Service ──写入──→ Redis 补偿队列 ──定时消费──→ CompensationService
```

### 服务发现

- 本地开发：硬编码服务地址（application-local.yml）
- 生产环境：通过配置或服务发现
