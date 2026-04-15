# 组件方法签名

> 注：详细业务规则将在构建阶段的功能设计中定义，此处仅定义方法签名和高层用途。

---

## 1. Auth Service

### 1.1 UserDomainService（领域服务）

```java
UserEntity register(String username, String password, String displayName, String role);
// 用途：注册新用户，密码 BCrypt 哈希存储

UserEntity getById(Long id);
// 用途：根据 ID 获取用户

UserEntity getByUsername(String username);
// 用途：根据用户名获取用户

PageResult<UserEntity> page(int page, int size, String keyword);
// 用途：分页查询用户列表，支持关键词搜索
```

### 1.2 AuthenticationDomainService（领域服务）

```java
AuthResult login(String username, String password);
// 用途：验证凭证，生成 JWT Token，返回用户信息
// 输出：AuthResult(token, userId, username, displayName, role)

TokenValidationResult validateToken(String token);
// 用途：验证 JWT Token 有效性
// 输出：TokenValidationResult(success, operatorId, message)

void checkLoginAttempts(String username);
// 用途：检查登录失败次数，超限则抛出异常

void recordFailedAttempt(String username);
// 用途：记录一次登录失败

void resetFailedAttempts(String username);
// 用途：登录成功后重置失败计数
```

### 1.3 UserRepository（仓储端口）

```java
UserEntity getById(Long id);
UserEntity getByUsername(String username);
PageResult<UserEntity> page(int page, int size, String keyword);
void save(UserEntity entity);
```

### 1.4 JwtSecurityService（安全端口）

```java
String generateToken(Long userId, String username, String role);
TokenClaims parseToken(String token);
```

---

## 2. Product Service

### 2.1 ProductDomainService（领域服务 — 补充方法）

```java
// 已有
ProductEntity getById(Long id);
PageResult<ProductEntity> page(int page, int size, String name, Long categoryId);
void create(...);

// 新增
void update(Long id, ...);
// 用途：更新商品信息

void delete(Long id);
// 用途：逻辑删除商品

void updateStatus(Long id, Integer status);
// 用途：上架/下架

void deductStock(Long productId, Integer quantity);
// 用途：扣减库存（乐观锁），供 Order Service 调用

void restoreStock(Long productId, Integer quantity);
// 用途：恢复库存，供订单取消时调用
```

### 2.2 CategoryDomainService（领域服务 — 新增）

```java
CategoryEntity getById(Long id);
// 用途：获取单个分类

List<CategoryEntity> getTree();
// 用途：获取完整分类树

List<CategoryEntity> getChildren(Long parentId);
// 用途：获取子分类列表

void create(String name, Long parentId, Integer sortOrder);
// 用途：创建分类

void update(Long id, String name, Integer sortOrder);
// 用途：更新分类

void delete(Long id);
// 用途：删除分类（有商品时拒绝）
```

### 2.3 CategoryRepository（仓储端口 — 新增）

```java
CategoryEntity getById(Long id);
List<CategoryEntity> getByParentId(Long parentId);
List<CategoryEntity> getAll();
void save(CategoryEntity entity);
void update(CategoryEntity entity);
void deleteById(Long id);
int countProductsByCategoryId(Long categoryId);
```

---

## 3. Points Service

### 3.1 PointsAccountDomainService（领域服务 — 新增）

```java
PointsAccountEntity getByUserId(Long userId);
// 用途：获取用户积分账户（不存在则自动创建）

void deduct(Long userId, Integer amount, String referenceType, String referenceId, String description);
// 用途：扣减积分（乐观锁），生成 SPEND 交易记录

void refund(Long userId, Integer amount, String referenceType, String referenceId, String description);
// 用途：退还积分，生成 REFUND 交易记录

void grant(Long userId, Integer amount, String description);
// 用途：发放积分，生成 EARN 交易记录

void batchGrant(List<Long> userIds, Integer amount, String description);
// 用途：批量发放积分
```

### 3.2 PointsTransactionDomainService（领域服务 — 新增）

```java
PageResult<PointsTransactionEntity> page(Long userId, int page, int size, String type);
// 用途：分页查询积分交易明细，支持按类型筛选
```

### 3.3 PointsAccountRepository（仓储端口 — 新增）

```java
PointsAccountEntity getByUserId(Long userId);
void save(PointsAccountEntity entity);
void update(PointsAccountEntity entity);
```

### 3.4 PointsTransactionRepository（仓储端口 — 新增）

```java
void save(PointsTransactionEntity entity);
PageResult<PointsTransactionEntity> page(Long userId, int page, int size, String type);
```

---

## 4. Order Service

### 4.1 OrderDomainService（领域服务 — 新增）

```java
OrderEntity create(Long userId, Long productId, String productName, String productImage,
                   Integer quantity, Integer pointsAmount);
// 用途：创建订单记录

OrderEntity getById(Long id);
// 用途：获取订单详情

PageResult<OrderEntity> pageByUser(Long userId, int page, int size);
// 用途：员工查询自己的订单

PageResult<OrderEntity> pageAll(int page, int size, String status, String keyword);
// 用途：管理员查询所有订单

void updateStatus(Long id, String newStatus);
// 用途：更新订单状态（含状态机校验）

void cancel(Long id, Long userId);
// 用途：取消订单（校验权限和状态）
```

### 4.2 RedemptionApplicationService（应用服务 — 新增）

```java
OrderEntity redeem(Long userId, Long productId, Integer quantity);
// 用途：编排兑换流程（校验→扣积分→扣库存→创建订单→失败补偿）
// 跨服务调用：Points Service + Product Service

void cancelOrder(Long orderId, Long userId);
// 用途：编排取消流程（取消订单→退还积分→恢复库存）
```

### 4.3 OrderRepository（仓储端口 — 新增）

```java
OrderEntity getById(Long id);
PageResult<OrderEntity> pageByUser(Long userId, int page, int size);
PageResult<OrderEntity> pageAll(int page, int size, String status, String keyword);
void save(OrderEntity entity);
void update(OrderEntity entity);
```

### 4.4 跨服务客户端（基础设施层 — 新增）

```java
// PointsServiceClient
PointsDeductResult deductPoints(Long userId, Integer amount, String orderNo);
void refundPoints(Long userId, Integer amount, String orderNo);

// ProductServiceClient
ProductInfo getProduct(Long productId);
StockDeductResult deductStock(Long productId, Integer quantity);
void restoreStock(Long productId, Integer quantity);
```

---

## 5. Frontend API 服务

### 5.1 authApi

```typescript
login(username: string, password: string): Promise<LoginResponse>
register(username: string, password: string, displayName: string): Promise<void>
getUserInfo(): Promise<UserInfo>
```

### 5.2 productApi

```typescript
getProducts(params: ProductListParams): Promise<PageResult<Product>>
getProductDetail(id: number): Promise<Product>
getCategories(): Promise<CategoryTree[]>
// Admin
createProduct(data: CreateProductRequest): Promise<void>
updateProduct(id: number, data: UpdateProductRequest): Promise<void>
deleteProduct(id: number): Promise<void>
updateProductStatus(id: number, status: number): Promise<void>
```

### 5.3 pointsApi

```typescript
getBalance(): Promise<PointsAccount>
getTransactions(params: TransactionListParams): Promise<PageResult<PointsTransaction>>
// Admin
grantPoints(userId: number, amount: number, description: string): Promise<void>
batchGrantPoints(userIds: number[], amount: number, description: string): Promise<BatchGrantResult>
```

### 5.4 orderApi

```typescript
redeem(productId: number, quantity: number): Promise<Order>
getMyOrders(params: OrderListParams): Promise<PageResult<Order>>
getOrderDetail(id: number): Promise<Order>
cancelOrder(id: number): Promise<void>
// Admin
getAllOrders(params: AdminOrderListParams): Promise<PageResult<Order>>
updateOrderStatus(id: number, status: string): Promise<void>
```

### 5.5 adminApi

```typescript
getUsers(params: UserListParams): Promise<PageResult<User>>
createUser(data: CreateUserRequest): Promise<void>
getDashboardStats(): Promise<DashboardStats>
getDashboardTrends(days: number): Promise<TrendData>
getDashboardRankings(): Promise<RankingData>
// Category
createCategory(data: CreateCategoryRequest): Promise<void>
updateCategory(id: number, data: UpdateCategoryRequest): Promise<void>
deleteCategory(id: number): Promise<void>
```
