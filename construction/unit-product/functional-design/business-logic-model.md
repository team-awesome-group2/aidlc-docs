# 业务逻辑模型 — unit-product

## 1. Product 领域服务（补充方法）

### 1.1 getById(Long id) — 已有

- 根据 ID 查询商品，不存在抛出 RESOURCE_NOT_FOUND
- 员工端调用时需额外校验 status=1（在应用层处理）

### 1.2 page(int page, int size, String name, Long categoryId) — 改造

- 分页查询商品列表
- 支持按名称模糊搜索
- 支持按 categoryId 筛选（改造：原 String category → Long categoryId）
- 当传入 categoryId 时，需查询该分类及其所有子孙分类的 ID 集合，按集合筛选
- 实现方式：先通过 CategoryRepository 获取子孙分类 ID 列表，再传入 ProductRepository 做 IN 查询

### 1.3 create(..., Long categoryId, ...) — 改造

- 创建商品，categoryId 替代原 category 字符串
- 校验 categoryId 对应的分类存在且状态为启用
- 校验 SKU 唯一性
- 新商品默认 status=0（未上架）、soldCount=0

### 1.4 update(Long id, ...) — 新增

- 更新商品信息
- SKU 不可修改（忽略传入的 SKU 值）
- 校验商品存在
- 如果修改了 categoryId，校验新分类存在且启用
- 使用乐观锁防止并发冲突

### 1.5 delete(Long id) — 新增

- 逻辑删除商品
- 校验商品存在
- 已删除的商品不影响历史订单（订单保留商品快照）

### 1.6 updateStatus(Long id, Integer status) — 新增

- 切换上架/下架状态
- 校验商品存在
- status 只能是 0 或 1

### 1.7 deductStock(Long productId, Integer quantity) — 新增

- 扣减库存（供 Order Service 内部接口调用）
- 校验商品存在
- 校验商品已上架（status=1），否则返回 PRODUCT_OFF_SHELF
- 校验库存充足（stock >= quantity），否则返回 STOCK_INSUFFICIENT
- 使用乐观锁更新：`SET stock = stock - quantity, sold_count = sold_count + quantity WHERE version = ?`
- 乐观锁冲突时内部自动重试，最多 3 次
- 3 次重试后仍失败则抛出异常

### 1.8 restoreStock(Long productId, Integer quantity) — 新增

- 恢复库存（订单取消时调用）
- 校验商品存在（即使已删除也应恢复库存——但逻辑删除的商品查不到，此处仅校验存在）
- 更新：`SET stock = stock + quantity, sold_count = sold_count - quantity`
- sold_count 不能小于 0（兜底保护）

---

## 2. Category 领域服务（全新）

### 2.1 getById(Long id)

- 根据 ID 查询分类，不存在抛出 RESOURCE_NOT_FOUND

### 2.2 getTree()

- 获取完整分类树
- 查询所有未删除的分类，在内存中组装树形结构
- 按 sortOrder 升序排列
- 返回结构：List&lt;CategoryTreeNode&gt;，每个节点包含 children 列表

### 2.3 getChildren(Long parentId)

- 获取指定父分类的直接子分类列表
- 按 sortOrder 升序排列

### 2.4 getDescendantIds(Long categoryId)

- 获取指定分类及其所有子孙分类的 ID 集合
- 用于商品列表的树形筛选
- 实现：查询所有分类，在内存中递归收集

### 2.5 create(String name, Long parentId, Integer sortOrder)

- 创建分类
- 如果 parentId 不为空，校验父分类存在
- 校验层级深度不超过 3 级
- 同一父分类下名称不能重复

### 2.6 update(Long id, String name, Integer sortOrder)

- 更新分类名称和排序
- 校验分类存在
- 不允许修改 parentId（不支持移动分类）

### 2.7 delete(Long id)

- 删除分类
- 校验分类存在
- 如果该分类下有商品（直接关联），拒绝删除
- 如果该分类有子分类：
  - 递归检查所有子孙分类下是否有商品
  - 任何子孙分类下有商品则拒绝删除
  - 所有子孙分类下都没有商品则级联删除所有子孙分类

---

## 3. 应用服务编排

### 3.1 ProductApplicationService（补充）

- `getDetail(Long id)` — 调用 ProductDomainService.getById()，转换为 DTO（含 categoryName）
- `list(ListProductRequest)` — 如果有 categoryId，先调用 CategoryDomainService.getDescendantIds() 获取 ID 集合，再调用 ProductDomainService.page()
- `create(CreateProductRequest)` — 校验 categoryId 有效后调用领域服务
- `update(UpdateProductRequest)` — 校验 categoryId 有效后调用领域服务
- `delete(Long id)` — 调用领域服务
- `updateStatus(Long id, Integer status)` — 调用领域服务
- `deductStock(Long productId, Integer quantity)` — 调用领域服务
- `restoreStock(Long productId, Integer quantity)` — 调用领域服务
- `getStats()` — 查询商品总数、上架数、热门商品排行

### 3.2 CategoryApplicationService（新增）

- `getTree()` — 调用 CategoryDomainService.getTree()
- `create(CreateCategoryRequest)` — 调用领域服务
- `update(UpdateCategoryRequest)` — 调用领域服务
- `delete(Long id)` — 调用领域服务

---

## 4. 接口层（Controller）

### 4.1 ProductController（补充端点）

| 端点 | 用途 | 权限 |
|------|------|------|
| POST /api/v1/public/product/list | 商品列表（员工端，仅上架） | 已认证 |
| POST /api/v1/public/product/detail | 商品详情 | 已认证 |
| POST /api/v1/admin/product/list | 商品列表（管理端，全部状态） | 管理员 |
| POST /api/v1/admin/product/create | 创建商品 | 管理员 |
| POST /api/v1/admin/product/update | 更新商品 | 管理员 |
| POST /api/v1/admin/product/delete | 删除商品 | 管理员 |
| POST /api/v1/admin/product/update-status | 上架/下架 | 管理员 |
| POST /api/v1/product/stats | 商品统计 | 管理员 |

### 4.2 CategoryController（新增）

| 端点 | 用途 | 权限 |
|------|------|------|
| POST /api/v1/public/category/tree | 获取分类树 | 已认证 |
| POST /api/v1/admin/category/create | 创建分类 | 管理员 |
| POST /api/v1/admin/category/update | 更新分类 | 管理员 |
| POST /api/v1/admin/category/delete | 删除分类 | 管理员 |

### 4.3 InternalProductController（新增）

| 端点 | 用途 | 调用方 |
|------|------|--------|
| POST /api/v1/internal/product/get | 查询商品信息 | Order Service |
| POST /api/v1/internal/product/deduct-stock | 库存扣减 | Order Service |
| POST /api/v1/internal/product/restore-stock | 库存恢复 | Order Service |

---

## 5. 统计接口实现方案

### POST /api/v1/product/stats

**热门商品排行（近 30 天兑换量）实现方案**：

Product Service 的 product 表只有累计 `soldCount`，没有按时间维度的兑换数据。实现近 30 天排行有两种路径：

**采用方案：基于 soldCount 累计排行 + 接口预留 days 参数**

- 当前阶段使用 `soldCount` 降序排列作为热门排行（因为系统刚上线，soldCount 本身就是近期数据）
- 统计接口响应结构与内部接口契约保持一致
- 未来如需精确的时间维度排行，可由 Order Service 的统计接口提供（Order Service 有订单创建时间）

**响应结构**：

```json
{
  "totalProducts": 156,
  "onShelfProducts": 120,
  "hotProducts": [
    { "productId": 1001, "name": "Apple AirPods Pro", "soldCount": 89 },
    { "productId": 1002, "name": "小米手环8", "soldCount": 76 }
  ]
}
```

- `totalProducts`：SELECT COUNT(*) FROM product WHERE deleted=0
- `onShelfProducts`：SELECT COUNT(*) FROM product WHERE deleted=0 AND status=1
- `hotProducts`：SELECT TOP 10 FROM product WHERE deleted=0 AND status=1 ORDER BY sold_count DESC

---

## 6. 可测试属性（PBT-01 合规）

### 6.1 Product 组件

| 属性 | 类别 | 描述 |
|------|------|------|
| 库存扣减/恢复 Round-trip | Round-trip | deductStock(n) 后 restoreStock(n)，stock 恢复原值 |
| 库存扣减 Invariant | Invariant | 扣减后 stock + soldCount = 原始 stock + 原始 soldCount（总量守恒） |
| 库存非负 Invariant | Invariant | 任何操作后 stock >= 0 |
| 状态切换 Idempotence | Idempotence | updateStatus(1) 连续调用两次，结果与一次相同 |
| 创建-查询 Round-trip | Round-trip | create 后 getById 返回的实体字段与输入一致 |

### 6.2 Category 组件

| 属性 | 类别 | 描述 |
|------|------|------|
| 树形结构 Invariant | Invariant | getTree() 返回的节点总数 = 数据库中未删除分类总数 |
| 层级深度 Invariant | Invariant | 树中任何节点的深度 <= 3 |
| 级联删除 Invariant | Invariant | 删除父分类后，其所有子孙分类也不存在 |
| 排序 Invariant | Invariant | getChildren() 返回的列表按 sortOrder 升序 |

### 6.3 无 PBT 属性的组件

- 统计接口（getStats）：纯查询聚合，无可逆操作或不变量，标记为 N/A
