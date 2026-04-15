# API Documentation

## Gateway Route Configuration

### Public Routes (无需认证)

| Method | Path | Target Service | Description |
|--------|------|---------------|-------------|
| ALL | `/api/v1/public/auth/**` | Auth :8001 | 认证公开接口 |
| ALL | `/api/v1/public/product/**` | Product :8002 | 商品公开接口 |
| ALL | `/api/v1/public/point/**` | Points :8003 | 积分公开接口 |
| ALL | `/api/v1/public/order/**` | Order :8004 | 订单公开接口 |

### Protected Routes (需要 JWT 认证)

| Method | Path | Target Service | Description |
|--------|------|---------------|-------------|
| ALL | `/api/v1/auth/**` | Auth :8001 | 认证受保护接口 |
| ALL | `/api/v1/product/**` | Product :8002 | 商品受保护接口 |
| ALL | `/api/v1/point/**` | Points :8003 | 积分受保护接口 |
| ALL | `/api/v1/order/**` | Order :8004 | 订单受保护接口 |

### Internal Routes

| Method | Path | Target Service | Description |
|--------|------|---------------|-------------|
| POST | `/api/v1/internal/auth/validate` | Auth :8001 | Gateway 内部 Token 验证 |

### Swagger/Docs Routes

| Path | Target | Description |
|------|--------|-------------|
| `/v3/api-docs/auth` | Auth :8001 | Auth 服务 API 文档 |
| `/v3/api-docs/product` | Product :8002 | Product 服务 API 文档 |
| `/v3/api-docs/point` | Points :8003 | Points 服务 API 文档 |
| `/v3/api-docs/order` | Order :8004 | Order 服务 API 文档 |

## REST APIs

### Auth Service (已实现 - 仅 Test CRUD)

#### POST /api/v1/public/test/get

- **Purpose**: 根据 ID 获取测试记录
- **Request**: `{ "id": Long }`
- **Response**: `Result<TestDTO>`

#### POST /api/v1/public/test/list

- **Purpose**: 分页查询测试记录
- **Request**: `{ "page": int, "size": int }`
- **Response**: `Result<PageResult<TestDTO>>`

#### POST /api/v1/public/test/create

- **Purpose**: 创建测试记录
- **Request**: `{ "name": String, "description": String }`
- **Response**: `Result<Void>`

#### POST /api/v1/public/test/update

- **Purpose**: 更新测试记录
- **Request**: `{ "id": Long, "name": String, "description": String }`
- **Response**: `Result<Void>`

#### POST /api/v1/public/test/delete

- **Purpose**: 删除测试记录
- **Request**: `{ "id": Long }`
- **Response**: `Result<Void>`

### Product Service (已实现)

#### POST /api/v1/public/product/create

- **Purpose**: 创建商品
- **Request**:

```json
{
  "name": "String (required, max 200)",
  "sku": "String (required, max 100, unique)",
  "category": "String (required, max 100)",
  "brand": "String (optional, max 100)",
  "pointsPrice": "Integer (required)",
  "marketPrice": "BigDecimal (optional)",
  "stock": "Integer (required)",
  "status": "Integer (optional)",
  "description": "String (optional)",
  "imageUrl": "String (optional, max 500)",
  "subtitle": "String (optional)",
  "deliveryMethod": "String (optional)",
  "serviceGuarantee": "String (optional)",
  "promotion": "String (optional)",
  "colors": "String (optional)",
  "specs": "List<Map<String,String>> (optional)"
}
```

- **Response**: `Result<Void>`
- **Validation**: `@NotBlank` on name/sku/category, `@NotNull` on pointsPrice/stock, `@Size` limits

#### POST /api/v1/public/product/list

- **Purpose**: 分页查询商品列表
- **Request**:

```json
{
  "page": "int (default 1)",
  "size": "int (default 20, max 100)",
  "name": "String (optional, fuzzy match)",
  "category": "String (optional, exact match)"
}
```

- **Response**: `Result<PageResult<ProductDTO>>`

### Gateway Internal API

#### Token Validation (Gateway → Auth)

- **Method**: POST
- **URL**: `http://localhost:8001/api/v1/internal/auth/validate`
- **Request**: `{ "token": "String" }`
- **Response**: `{ "success": boolean, "operatorId": "String", "message": "String" }`
- **Status**: 尚未在 Auth Service 中实现

## Internal APIs (Domain Service Interfaces)

### ProductDomainService

```java
ProductEntity getById(Long id);
PageResult<ProductEntity> page(int page, int size, String name, String category);
void create(String name, String sku, String category, String brand,
            Integer pointsPrice, BigDecimal marketPrice, Integer stock,
            Integer status, String description, String imageUrl,
            String subtitle, String deliveryMethod, String serviceGuarantee,
            String promotion, String colors, List<Map<String, String>> specs);
```

### AuthenticationService (Gateway Domain)

```java
Mono<AuthenticationResult> validate(String token);
```

### EncryptionService (Security Port)

```java
String encrypt(String plainText);
String decrypt(String cipherText);
```

## Data Models

### ProductEntity

| Field | Type | Description |
|-------|------|-------------|
| id | Long | 主键 |
| name | String | 商品名称 |
| sku | String | 商品编号 (唯一) |
| category | String | 分类 |
| brand | String | 品牌 |
| pointsPrice | Integer | 积分价格 |
| marketPrice | BigDecimal | 市场参考价 |
| stock | Integer | 库存 |
| soldCount | Integer | 已兑换数量 |
| status | Integer | 状态 (0=下架, 1=上架) |
| description | String | 描述 |
| imageUrl | String | 主图 URL |
| subtitle | String | 副标题 |
| deliveryMethod | String | 配送方式 |
| serviceGuarantee | String | 服务保障 |
| promotion | String | 促销活动 |
| colors | String | 可选颜色 |
| specs | List<Map> | 规格参数 (JSON) |

### AuthenticationResult (Gateway Value Object)

| Field | Type | Description |
|-------|------|-------------|
| authenticated | boolean | 是否认证成功 |
| operatorId | String | 操作员 ID |
| message | String | 消息 |

### TokenInfo (Gateway Value Object)

| Field | Type | Description |
|-------|------|-------------|
| token | String | JWT Token (从 Bearer header 解析) |

### TestEntity (占位)

| Field | Type | Description |
|-------|------|-------------|
| id | Long | 主键 |
| name | String | 名称 |
| description | String | 描述 |
| createdAt | LocalDateTime | 创建时间 |
| updatedAt | LocalDateTime | 更新时间 |

## Unified Response Format

```java
public class Result<T> {
    private int code;       // 业务状态码
    private String message; // 消息
    private T data;         // 数据
}
```

## API Design Convention

- 所有 API 统一使用 POST 方法 + JSON Body
- 公开接口路径: `/api/v1/public/{service}/...`
- 受保护接口路径: `/api/v1/{service}/...`
- 内部接口路径: `/api/v1/internal/{service}/...`
