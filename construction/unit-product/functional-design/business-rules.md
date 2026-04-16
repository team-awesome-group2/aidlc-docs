# 业务规则 — unit-product

## 1. 商品规则

### BR-PROD-01: SKU 唯一性

- 创建商品时 SKU 必须全局唯一（含已逻辑删除的记录）
- 违反时返回错误码 `CONFLICT_001`，消息"SKU已存在: {sku}"

### BR-PROD-02: SKU 不可变

- 商品创建后 SKU 不允许修改
- 更新接口忽略传入的 SKU 值（不报错，静默忽略）

### BR-PROD-03: categoryId 必填且有效

- 创建和更新商品时 categoryId 不允许为空
- categoryId 必须对应一个存在且启用（status=1）的分类
- 违反时返回错误码 `PARAM_001`，消息"分类不存在或已禁用"

### BR-PROD-04: 商品状态值域

- status 只能是 0（已下架）或 1（已上架）
- 新创建商品默认 status=0
- 违反时返回错误码 `PARAM_001`

### BR-PROD-05: 积分价格正数

- pointsPrice 必须 > 0
- 违反时返回参数校验错误

### BR-PROD-06: 库存非负

- stock 不允许为负数
- 创建时 stock 默认为 0

### BR-PROD-07: 逻辑删除

- 删除操作为逻辑删除（deleted=0 → deleted=1）
- 已删除商品不影响历史订单
- 已删除商品不出现在任何查询结果中（MyBatis-Plus @TableLogic 自动过滤）

### BR-PROD-08: 乐观锁

- 更新和库存操作使用 version 字段做乐观锁
- 并发冲突时抛出异常（库存扣减有重试机制，见 BR-STOCK-02）

---

## 2. 库存规则

### BR-STOCK-01: 库存扣减前置校验

- 商品必须存在（否则 RESOURCE_NOT_FOUND）
- 商品必须已上架 status=1（否则 PRODUCT_OFF_SHELF）
- 库存必须充足 stock >= quantity（否则 STOCK_INSUFFICIENT）
- quantity 必须 > 0

### BR-STOCK-02: 库存扣减乐观锁重试

- 使用乐观锁 SQL：`UPDATE product SET stock = stock - ?, sold_count = sold_count + ?, version = version + 1 WHERE id = ? AND version = ? AND stock >= ?`
- 乐观锁冲突时自动重试，最多 3 次
- 每次重试前重新读取最新数据，重新校验前置条件
- 3 次重试后仍失败则抛出 `STOCK_INSUFFICIENT` 或系统异常

### BR-STOCK-03: 库存恢复

- 商品必须存在（否则 RESOURCE_NOT_FOUND）
- 恢复操作：`stock = stock + quantity, sold_count = sold_count - quantity`
- sold_count 兜底保护：如果 sold_count - quantity < 0，则 sold_count 设为 0
- quantity 必须 > 0

### BR-STOCK-04: 库存守恒

- 对于任意商品：`stock + soldCount` 的总量在扣减/恢复操作前后保持不变
- 这是系统不变量，可通过 PBT 验证

---

## 3. 分类规则

### BR-CAT-01: 最大层级深度

- 分类树最多 3 级
- 创建分类时校验：
  - parentId 为空 → 顶级分类（第 1 级），允许
  - parentId 指向顶级分类 → 第 2 级，允许
  - parentId 指向第 2 级分类 → 第 3 级，允许
  - parentId 指向第 3 级分类 → 第 4 级，拒绝
- 校验方式：沿 parentId 链向上追溯，计算目标层级
- 违反时返回错误码 `BIZ_001`，消息"分类层级不能超过3级"

### BR-CAT-02: 同级名称唯一

- 同一 parentId 下分类名称不能重复
- 违反时返回错误码 `CONFLICT_001`，消息"同级分类名称已存在: {name}"

### BR-CAT-03: 删除约束

- 删除分类前检查：
  1. 该分类直接关联的商品数量（`SELECT COUNT(*) FROM product WHERE category_id = ? AND deleted = 0`）
  2. 递归检查所有子孙分类关联的商品数量
- 如果任何层级有商品关联，拒绝删除，返回 `BIZ_002`，消息"该分类或其子分类下有商品，无法删除"
- 如果无商品关联，级联逻辑删除该分类及所有子孙分类

### BR-CAT-04: parentId 不可变

- 分类创建后不允许修改 parentId（不支持移动分类到其他父节点）
- 更新接口仅允许修改 name 和 sortOrder

### BR-CAT-05: 分类状态

- status：0=禁用，1=启用，默认 1
- 禁用的分类不能被商品关联（BR-PROD-03）

---

## 4. 查询规则

### BR-QUERY-01: 员工端商品列表

- 仅返回 status=1（已上架）的商品
- 支持按名称模糊搜索
- 支持按 categoryId 筛选（包含子孙分类的商品）
- 分页：默认 page=1, size=20, 最大 size=100

### BR-QUERY-02: 管理端商品列表

- 返回所有状态的商品（上架 + 下架）
- 支持按名称模糊搜索和 categoryId 筛选
- 分页参数同上

### BR-QUERY-03: 分类树查询

- 返回所有未删除、已启用的分类
- 组装为树形结构，按 sortOrder 升序排列
- 员工端和管理端共用同一接口

---

## 5. 内部接口规则

### BR-INTERNAL-01: 内部接口无权限校验

- `/api/v1/internal/` 路径下的接口不做用户权限校验
- 这些接口仅供其他微服务调用，由 Gateway 路由规则保护（不对外暴露）

### BR-INTERNAL-02: 内部接口响应格式

- 使用统一 `Result<T>` 格式（code, message, data）
- 错误码与内部接口契约文档一致：
  - `SUCCESS` — 操作成功
  - `RESOURCE_NOT_FOUND` — 商品不存在
  - `STOCK_INSUFFICIENT` — 库存不足
  - `PRODUCT_OFF_SHELF` — 商品已下架

---

## 6. 错误码汇总

| 错误码 | HTTP 状态码 | 场景 | 使用方 |
|--------|-----------|------|--------|
| NOT_FOUND_001 | 404 | 商品/分类不存在 | Product/Category |
| CONFLICT_001 | 409 | SKU 已存在 / 同级分类名称重复 | Product/Category |
| PARAM_001 | 400 | 参数校验失败（categoryId 无效、status 非法等） | Product/Category |
| BIZ_001 | 200 | 分类层级超过 3 级 | Category |
| BIZ_002 | 200 | 分类下有商品无法删除 | Category |
| STOCK_INSUFFICIENT | 200 | 库存不足（内部接口） | Product Internal |
| PRODUCT_OFF_SHELF | 200 | 商品已下架（内部接口） | Product Internal |

注：内部接口错误码（STOCK_INSUFFICIENT、PRODUCT_OFF_SHELF）不走 GlobalExceptionHandler 的前缀映射，直接在 Result 中返回字符串 code。需要在 SampleErrorCode 或新建 ProductErrorCode 枚举中定义。
