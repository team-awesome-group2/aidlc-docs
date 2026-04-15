# 内部接口契约文档

> 本文档定义所有跨服务调用的 HTTP 接口契约。各服务开发者按此契约独立开发，无需额外沟通接口格式。
>
> **通用约定**：
>
> - 所有接口使用 POST 方法 + JSON Body（与现有风格一致）
> - 统一返回格式：`Result<T>`（code: String, message: String, data: T）
> - 成功时 code = "SUCCESS"，失败时 code = 具体错误码
> - 内部接口路径前缀：`/api/v1/internal/{service}/`
> - operatorId 由 Gateway 通过 Header `X-Operator-Id` 注入，内部接口也可从 Header 获取

---

## 1. Auth Service → Gateway（Token 验证）

> ⚠️ 此接口 Gateway 侧已有现成代码（AuthServiceClient + DTO），Auth Service 必须严格按此契约实现。

### POST /api/v1/internal/auth/validate

**调用方**：Gateway（AuthServiceClient via WebClient）
**提供方**：Auth Service

**请求体**：

```json
{
  "token": "eyJhbGciOiJIUzI1NiJ9..."
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| token | String | 是 | JWT Token（不含 "Bearer " 前缀，Gateway 已剥离） |

**响应体**：

```json
{
  "success": true,
  "operatorId": "12345",
  "message": null
}
```

| 字段 | 类型 | 说明 |
|------|------|------|
| success | boolean | 验证是否成功 |
| operatorId | String | 用户 ID（success=true 时必填） |
| message | String | 失败原因（success=false 时必填） |

**注意**：此接口不使用 `Result<T>` 包装，直接返回扁平 JSON。因为 Gateway 侧 `AuthValidateResponse` 已定义为扁平结构。

**失败场景**：

| 场景 | success | message |
|------|---------|---------|
| Token 有效 | true | null |
| Token 已过期 | false | "Token已过期" |
| Token 签名无效 | false | "Token无效" |
| Token 格式错误 | false | "Token格式错误" |

---

## 2. Product Service → Order Service（商品查询 + 库存管理）

### 2.1 POST /api/v1/internal/product/get

**调用方**：Order Service（ProductServiceClient）
**提供方**：Product Service

**用途**：兑换前查询商品信息（价格、库存、状态）

**请求体**：

```json
{
  "productId": 1001
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| productId | Long | 是 | 商品 ID |

**响应体**：

```json
{
  "code": "SUCCESS",
  "message": "操作成功",
  "data": {
    "id": 1001,
    "name": "Apple AirPods Pro",
    "sku": "APD-PRO-001",
    "pointsPrice": 2000,
    "marketPrice": 1799.00,
    "stock": 50,
    "status": 1,
    "imageUrl": "https://example.com/airpods.jpg"
  }
}
```

**data 字段说明**：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | Long | 商品 ID |
| name | String | 商品名称（订单需要快照） |
| sku | String | 商品编号 |
| pointsPrice | Integer | 积分价格（用于计算兑换所需积分） |
| marketPrice | BigDecimal | 市场参考价 |
| stock | Integer | 当前库存（用于校验库存是否充足） |
| status | Integer | 状态：0=已下架，1=已上架（用于校验商品是否可兑换） |
| imageUrl | String | 主图 URL（订单需要快照） |

**错误码**：

| code | 场景 |
|------|------|
| SUCCESS | 查询成功 |
| RESOURCE_NOT_FOUND | 商品不存在或已删除 |

---

### 2.2 POST /api/v1/internal/product/deduct-stock

**调用方**：Order Service（ProductServiceClient）
**提供方**：Product Service

**用途**：兑换时扣减库存

**请求体**：

```json
{
  "productId": 1001,
  "quantity": 1
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| productId | Long | 是 | 商品 ID |
| quantity | Integer | 是 | 扣减数量（必须 > 0） |

**响应体（成功）**：

```json
{
  "code": "SUCCESS",
  "message": "操作成功",
  "data": {
    "productId": 1001,
    "remainingStock": 49
  }
}
```

**data 字段说明**：

| 字段 | 类型 | 说明 |
|------|------|------|
| productId | Long | 商品 ID |
| remainingStock | Integer | 扣减后剩余库存 |

**错误码**：

| code | 场景 |
|------|------|
| SUCCESS | 扣减成功 |
| STOCK_INSUFFICIENT | 库存不足 |
| RESOURCE_NOT_FOUND | 商品不存在 |
| PRODUCT_OFF_SHELF | 商品已下架 |

---

### 2.3 POST /api/v1/internal/product/restore-stock

**调用方**：Order Service（ProductServiceClient）
**提供方**：Product Service

**用途**：订单取消时恢复库存

**请求体**：

```json
{
  "productId": 1001,
  "quantity": 1
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| productId | Long | 是 | 商品 ID |
| quantity | Integer | 是 | 恢复数量（必须 > 0） |

**响应体（成功）**：

```json
{
  "code": "SUCCESS",
  "message": "操作成功",
  "data": {
    "productId": 1001,
    "remainingStock": 50
  }
}
```

**错误码**：

| code | 场景 |
|------|------|
| SUCCESS | 恢复成功 |
| RESOURCE_NOT_FOUND | 商品不存在 |

---

## 3. Points Service → Order Service（积分扣减 + 退还）

### 3.1 POST /api/v1/internal/point/deduct

**调用方**：Order Service（PointsServiceClient）
**提供方**：Points Service

**用途**：兑换时扣减用户积分

**请求体**：

```json
{
  "userId": 12345,
  "amount": 2000,
  "referenceType": "ORDER",
  "referenceId": "ORD20260415001",
  "description": "兑换商品: Apple AirPods Pro"
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| userId | Long | 是 | 用户 ID |
| amount | Integer | 是 | 扣减积分数（必须 > 0） |
| referenceType | String | 是 | 关联业务类型，固定传 "ORDER" |
| referenceId | String | 是 | 关联业务 ID（订单号） |
| description | String | 否 | 描述信息 |

**响应体（成功）**：

```json
{
  "code": "SUCCESS",
  "message": "操作成功",
  "data": {
    "userId": 12345,
    "remainingBalance": 580,
    "transactionId": 9001
  }
}
```

**data 字段说明**：

| 字段 | 类型 | 说明 |
|------|------|------|
| userId | Long | 用户 ID |
| remainingBalance | Integer | 扣减后剩余余额 |
| transactionId | Long | 本次交易记录 ID（可用于对账） |

**错误码**：

| code | 场景 |
|------|------|
| SUCCESS | 扣减成功 |
| POINTS_INSUFFICIENT | 积分余额不足 |
| USER_NOT_FOUND | 用户积分账户不存在 |

---

### 3.2 POST /api/v1/internal/point/refund

**调用方**：Order Service（PointsServiceClient）
**提供方**：Points Service

**用途**：订单取消时退还积分

**请求体**：

```json
{
  "userId": 12345,
  "amount": 2000,
  "referenceType": "ORDER_CANCEL",
  "referenceId": "ORD20260415001",
  "description": "订单取消退还: ORD20260415001"
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| userId | Long | 是 | 用户 ID |
| amount | Integer | 是 | 退还积分数（必须 > 0） |
| referenceType | String | 是 | 关联业务类型，固定传 "ORDER_CANCEL" |
| referenceId | String | 是 | 关联业务 ID（原订单号） |
| description | String | 否 | 描述信息 |

**响应体（成功）**：

```json
{
  "code": "SUCCESS",
  "message": "操作成功",
  "data": {
    "userId": 12345,
    "remainingBalance": 2580,
    "transactionId": 9002
  }
}
```

**错误码**：

| code | 场景 |
|------|------|
| SUCCESS | 退还成功 |
| USER_NOT_FOUND | 用户积分账户不存在 |

---

## 4. 统计 API（仪表盘数据）

> 仪表盘数据分散在多个服务中，采用**各服务各自暴露统计接口，前端分别调用**的方案。
> 理由：避免引入聚合层，保持服务独立性，前端并行请求性能可接受。

### 4.1 POST /api/v1/product/stats

**提供方**：Product Service
**调用方**：Frontend（通过 Gateway）

**请求体**：

```json
{}
```

**响应体**：

```json
{
  "code": "SUCCESS",
  "message": "操作成功",
  "data": {
    "totalProducts": 156,
    "onShelfProducts": 120,
    "hotProducts": [
      { "productId": 1001, "name": "Apple AirPods Pro", "soldCount": 89 },
      { "productId": 1002, "name": "小米手环8", "soldCount": 76 }
    ]
  }
}
```

### 4.2 POST /api/v1/auth/stats

**提供方**：Auth Service
**调用方**：Frontend（通过 Gateway）

**请求体**：

```json
{}
```

**响应体**：

```json
{
  "code": "SUCCESS",
  "message": "操作成功",
  "data": {
    "totalUsers": 500,
    "totalEmployees": 480,
    "totalAdmins": 20
  }
}
```

### 4.3 POST /api/v1/order/stats

**提供方**：Order Service
**调用方**：Frontend（通过 Gateway）

**请求体**：

```json
{
  "days": 30
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| days | Integer | 否 | 趋势天数，默认 30 |

**响应体**：

```json
{
  "code": "SUCCESS",
  "message": "操作成功",
  "data": {
    "monthlyRedemptions": 342,
    "trends": [
      { "date": "2026-04-01", "redemptions": 12 },
      { "date": "2026-04-02", "redemptions": 15 }
    ]
  }
}
```

### 4.4 POST /api/v1/point/stats

**提供方**：Points Service
**调用方**：Frontend（通过 Gateway）

**请求体**：

```json
{
  "days": 30
}
```

**响应体**：

```json
{
  "code": "SUCCESS",
  "message": "操作成功",
  "data": {
    "totalCirculation": 1250000,
    "monthlyEarned": 50000,
    "monthlySpent": 38000,
    "trends": [
      { "date": "2026-04-01", "earned": 1500, "spent": 1200 },
      { "date": "2026-04-02", "earned": 1800, "spent": 1400 }
    ],
    "activeUsers": [
      { "userId": 12345, "displayName": "李明", "totalSpent": 5600 },
      { "userId": 12346, "displayName": "张三", "totalSpent": 4200 }
    ]
  }
}
```

---

## 5. 接口汇总

### 跨服务内部接口（服务间直接调用）

| # | 接口 | 提供方 | 调用方 | 用途 |
|---|------|--------|--------|------|
| 1 | POST /api/v1/internal/auth/validate | Auth | Gateway | Token 验证 |
| 2 | POST /api/v1/internal/product/get | Product | Order | 查询商品信息 |
| 3 | POST /api/v1/internal/product/deduct-stock | Product | Order | 扣减库存 |
| 4 | POST /api/v1/internal/product/restore-stock | Product | Order | 恢复库存 |
| 5 | POST /api/v1/internal/point/deduct | Points | Order | 扣减积分 |
| 6 | POST /api/v1/internal/point/refund | Points | Order | 退还积分 |

### 统计接口（前端通过 Gateway 调用）

| # | 接口 | 提供方 | 用途 |
|---|------|--------|------|
| 7 | POST /api/v1/auth/stats | Auth | 用户统计 |
| 8 | POST /api/v1/product/stats | Product | 商品统计 + 热门排行 |
| 9 | POST /api/v1/order/stats | Order | 订单统计 + 趋势 |
| 10 | POST /api/v1/point/stats | Points | 积分统计 + 趋势 + 活跃用户 |

### 错误码汇总

| 错误码 | 含义 | 使用服务 |
|--------|------|---------|
| SUCCESS | 操作成功 | 全部 |
| RESOURCE_NOT_FOUND | 资源不存在 | Product |
| STOCK_INSUFFICIENT | 库存不足 | Product |
| PRODUCT_OFF_SHELF | 商品已下架 | Product |
| POINTS_INSUFFICIENT | 积分余额不足 | Points |
| USER_NOT_FOUND | 用户不存在 | Points, Auth |

---

## 6. 开发者速查

### 开发者 A（Auth Service）需要实现

- `POST /api/v1/internal/auth/validate` — 严格按 Gateway 已有 DTO 实现
- `POST /api/v1/auth/stats` — 用户统计

### 开发者 B（Product Service）需要实现

- `POST /api/v1/internal/product/get` — 商品查询（供 Order 调用）
- `POST /api/v1/internal/product/deduct-stock` — 库存扣减
- `POST /api/v1/internal/product/restore-stock` — 库存恢复
- `POST /api/v1/product/stats` — 商品统计 + 热门排行

### 开发者 C（Points Service）需要实现

- `POST /api/v1/internal/point/deduct` — 积分扣减
- `POST /api/v1/internal/point/refund` — 积分退还
- `POST /api/v1/point/stats` — 积分统计 + 趋势 + 活跃用户

### 开发者 D（Order Service）需要调用

- ProductServiceClient：调用接口 2、3、4
- PointsServiceClient：调用接口 5、6
- 自身实现：`POST /api/v1/order/stats` — 订单统计 + 趋势
