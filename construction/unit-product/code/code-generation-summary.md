# 代码生成摘要 — unit-product

## 生成统计

- **修改文件**：8 个
- **新增文件**：25 个
- **Flyway 迁移**：1 个（V3）

## 修改的文件

| 文件 | 变更内容 |
|------|---------|
| ProductEntity.java | `String category` → `Long categoryId` |
| ProductPO.java | `String category` → `Long categoryId` |
| ProductDTO.java | `String category` → `Long categoryId` + 新增 `categoryName` |
| CreateProductRequest.java | `String category` → `Long categoryId` |
| ListProductRequest.java | `String category` → `Long categoryId` |
| ProductDomainService.java | 新增 update/delete/updateStatus/deductStock/restoreStock 等方法 |
| ProductDomainServiceImpl.java | 实现所有新增方法，含乐观锁重试 |
| ProductApplicationService.java | 新增全部方法签名 |
| ProductApplicationServiceImpl.java | 实现全部方法，集成缓存和分类校验 |
| ProductController.java | 补充 detail/admin 端点，调整路径 |
| ProductRepository.java | 新增 deductStock/restoreStock/countByCategoryId 等方法 |
| ProductRepositoryImpl.java | 实现新增方法，Entity↔PO 转换改造 |
| ProductMapper.java | 新增方法签名 |
| ProductMapper.xml | 分页查询改为 categoryIds IN，新增统计/库存 SQL |

## 新增的文件

### 领域层
- CategoryEntity.java
- CategoryDomainService.java
- CategoryDomainServiceImpl.java
- CategoryRepository.java
- ProductCacheService.java（缓存端口）
- CategoryCacheService.java（缓存端口）

### 基础设施层
- CategoryPO.java
- CategoryMapper.java
- CategoryMapper.xml
- CategoryRepositoryImpl.java
- ProductCacheServiceImpl.java（Redis）
- CategoryCacheServiceImpl.java（Redis）

### 应用层
- CategoryDTO.java
- CategoryTreeNode.java
- CreateCategoryRequest.java
- UpdateCategoryRequest.java
- DeleteCategoryRequest.java
- CategoryApplicationService.java
- CategoryApplicationServiceImpl.java
- GetProductRequest.java
- UpdateProductRequest.java
- DeleteProductRequest.java
- UpdateStatusRequest.java
- DeductStockRequest.java
- RestoreStockRequest.java
- ProductStatsDTO.java

### 接口层
- CategoryController.java
- InternalProductController.java

### 数据库
- V3__create_category_and_modify_product.sql

### 基础
- ProductErrorCode.java

## 故事覆盖

| 故事 | 状态 |
|------|------|
| US-PROD-01 浏览商品列表 | ✅ |
| US-PROD-02 查看商品详情 | ✅ |
| US-PROD-03 创建商品 | ✅ |
| US-PROD-04 编辑商品 | ✅ |
| US-PROD-05 删除商品 | ✅ |
| US-PROD-06 商品上架/下架 | ✅ |
| US-PROD-07 管理商品分类 | ✅ |
| US-PROD-08 库存扣减 | ✅ |
| US-PROD-09 库存恢复 | ✅ |

## API 端点汇总

| 端点 | 用途 |
|------|------|
| POST /api/v1/public/product/list | 商品列表（员工端） |
| POST /api/v1/public/product/detail | 商品详情 |
| POST /api/v1/admin/product/list | 商品列表（管理端） |
| POST /api/v1/admin/product/create | 创建商品 |
| POST /api/v1/admin/product/update | 更新商品 |
| POST /api/v1/admin/product/delete | 删除商品 |
| POST /api/v1/admin/product/update-status | 上架/下架 |
| POST /api/v1/product/stats | 商品统计 |
| POST /api/v1/public/category/tree | 分类树 |
| POST /api/v1/admin/category/create | 创建分类 |
| POST /api/v1/admin/category/update | 更新分类 |
| POST /api/v1/admin/category/delete | 删除分类 |
| POST /api/v1/internal/product/get | 商品查询（内部） |
| POST /api/v1/internal/product/deduct-stock | 库存扣减（内部） |
| POST /api/v1/internal/product/restore-stock | 库存恢复（内部） |
