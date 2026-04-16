# 集成测试指令 — unit-product

## 目的

验证 Product Service 的内部接口与 Order Service 的交互契约正确性。

## 前置条件

- Product Service 已启动（端口 8002）
- MySQL 和 Redis 已启动

## 测试场景

### 场景 1：内部接口 — 商品查询

```bash
# 先创建一个商品（需要先创建分类）
curl -X POST http://localhost:8002/api/v1/admin/category/create \
  -H "Content-Type: application/json" \
  -d '{"name":"数码电子","sortOrder":1}'

curl -X POST http://localhost:8002/api/v1/admin/product/create \
  -H "Content-Type: application/json" \
  -d '{"name":"测试商品","sku":"TEST-001","categoryId":1,"pointsPrice":100,"stock":50}'

# 内部接口查询
curl -X POST http://localhost:8002/api/v1/internal/product/get \
  -H "Content-Type: application/json" \
  -d '{"id":1}'
```

预期：返回 `{"code":"SUCCESS","data":{"id":1,"name":"测试商品",...}}`

### 场景 2：内部接口 — 库存扣减

```bash
curl -X POST http://localhost:8002/api/v1/internal/product/deduct-stock \
  -H "Content-Type: application/json" \
  -d '{"productId":1,"quantity":5}'
```

预期：返回 `{"code":"SUCCESS","data":{"productId":1,"remainingStock":45}}`

### 场景 3：内部接口 — 库存恢复

```bash
curl -X POST http://localhost:8002/api/v1/internal/product/restore-stock \
  -H "Content-Type: application/json" \
  -d '{"productId":1,"quantity":5}'
```

预期：返回 `{"code":"SUCCESS","data":{"productId":1,"remainingStock":50}}`

### 场景 4：内部接口 — 下架商品扣减失败

```bash
# 先下架商品
curl -X POST http://localhost:8002/api/v1/admin/product/update-status \
  -H "Content-Type: application/json" \
  -d '{"id":1,"status":0}'

# 尝试扣减
curl -X POST http://localhost:8002/api/v1/internal/product/deduct-stock \
  -H "Content-Type: application/json" \
  -d '{"productId":1,"quantity":1}'
```

预期：返回错误 `{"code":"BIZ_002","message":"商品已下架"}`

### 场景 5：分类树形筛选

```bash
# 创建子分类
curl -X POST http://localhost:8002/api/v1/admin/category/create \
  -H "Content-Type: application/json" \
  -d '{"name":"手机","parentId":1,"sortOrder":1}'

# 按父分类筛选（应包含子分类商品）
curl -X POST http://localhost:8002/api/v1/public/product/list \
  -H "Content-Type: application/json" \
  -d '{"categoryId":1}'
```

预期：返回父分类及所有子分类下的商品

### 场景 6：分类级联删除

```bash
# 创建无商品的分类树
curl -X POST http://localhost:8002/api/v1/admin/category/create \
  -H "Content-Type: application/json" \
  -d '{"name":"待删除分类","sortOrder":99}'

curl -X POST http://localhost:8002/api/v1/admin/category/create \
  -H "Content-Type: application/json" \
  -d '{"name":"待删除子分类","parentId":3,"sortOrder":1}'

# 删除父分类（应级联删除子分类）
curl -X POST http://localhost:8002/api/v1/admin/category/delete \
  -H "Content-Type: application/json" \
  -d '{"id":3}'
```

预期：父分类和子分类都被删除

## 清理

测试完成后清理测试数据：

```bash
# 停止服务
lsof -ti:8002 | xargs kill -9

# 如需重置数据库
mysql -h localhost -P 3307 -u root -proot -e "DROP DATABASE awsome_shop_product; CREATE DATABASE awsome_shop_product;"
```
