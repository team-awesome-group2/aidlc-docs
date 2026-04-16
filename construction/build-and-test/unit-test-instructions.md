# 单元测试指令 — unit-product

## 测试框架

- JUnit 5 + Mockito（单元测试）
- jqwik（属性测试 PBT）
- Spring Boot Test（集成测试）

## 运行单元测试

```bash
cd awsome-shop-product-service
mvn test
```

## 测试覆盖范围

### 领域服务单元测试（Mock Repository/Cache）

#### ProductDomainServiceImpl 测试用例
1. `getById` — 商品存在返回实体
2. `getById` — 商品不存在抛出 RESOURCE_NOT_FOUND
3. `create` — 正常创建成功
4. `create` — SKU 重复抛出 RESOURCE_ALREADY_EXISTS
5. `update` — 正常更新（SKU 不变）
6. `delete` — 逻辑删除成功
7. `updateStatus` — 状态切换成功
8. `deductStock` — 正常扣减成功
9. `deductStock` — 商品已下架抛出 PRODUCT_OFF_SHELF
10. `deductStock` — 库存不足抛出 STOCK_INSUFFICIENT
11. `deductStock` — 乐观锁冲突重试成功
12. `deductStock` — 3 次重试后仍失败
13. `restoreStock` — 正常恢复成功

#### CategoryDomainServiceImpl 测试用例
1. `create` — 顶级分类创建成功
2. `create` — 二级分类创建成功
3. `create` — 超过 3 级抛出 CATEGORY_DEPTH_EXCEEDED
4. `create` — 同级名称重复抛出 CATEGORY_NAME_DUPLICATE
5. `update` — 正常更新成功
6. `delete` — 无商品无子分类删除成功
7. `delete` — 有子分类无商品级联删除
8. `delete` — 有商品抛出 CATEGORY_HAS_PRODUCTS
9. `getDescendantIds` — 返回自身及所有子孙 ID

### PBT 属性测试（jqwik）

#### Product PBT
1. **库存扣减/恢复 Round-trip**：deductStock(n) + restoreStock(n) → stock 恢复原值
2. **库存守恒 Invariant**：扣减后 stock + soldCount = 原始总量
3. **库存非负 Invariant**：任何操作后 stock >= 0
4. **状态切换 Idempotence**：updateStatus(1) 连续两次 = 一次

#### Category PBT
5. **层级深度 Invariant**：树中任何节点深度 <= 3
6. **排序 Invariant**：getChildren() 按 sortOrder 升序

### 集成测试（Spring Boot Test + 本地 MySQL）

#### API 集成测试
1. POST /api/v1/admin/product/create — 创建商品全链路
2. POST /api/v1/public/product/list — 分页查询（含分类筛选）
3. POST /api/v1/public/product/detail — 商品详情（含缓存）
4. POST /api/v1/admin/product/update — 更新商品
5. POST /api/v1/admin/product/delete — 删除商品
6. POST /api/v1/admin/product/update-status — 上下架
7. POST /api/v1/internal/product/deduct-stock — 库存扣减
8. POST /api/v1/internal/product/restore-stock — 库存恢复
9. POST /api/v1/public/category/tree — 分类树
10. POST /api/v1/admin/category/create — 创建分类
11. POST /api/v1/admin/category/delete — 删除分类（级联）
12. POST /api/v1/product/stats — 商品统计

## 测试配置

测试使用 `application-test.yml` 配置，连接本地 MySQL 测试库。确保测试前数据库已创建且 Flyway 迁移已执行。
