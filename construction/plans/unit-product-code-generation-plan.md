# 代码生成计划 — 单元 2: unit-product（商品服务）

## 单元上下文

- **仓库**：awsome-shop-product-service
- **关联故事**：US-PROD-01~09（9 个）
- **依赖**：无外部服务依赖
- **被依赖**：Order Service（内部接口：商品查询、库存扣减/恢复）、Frontend（API）
- **架构**：DDD + 六边形架构，Maven 多模块

## 变更概览

- **修改文件**：~10 个（ProductEntity、ProductPO、ProductDTO 等 category→categoryId 改造）
- **新增文件**：~20 个（Category 全套 + 内部接口 + 缓存 + 错误码 + Flyway）
- **Flyway 迁移**：V3（创建 category 表 + 修改 product 表）

---

## 执行步骤

### Step 1: Flyway 数据库迁移脚本
- [x] 创建 `bootstrap/src/main/resources/db/migration/V3__create_category_and_modify_product.sql`
- 覆盖故事：US-PROD-07（分类管理基础）、US-PROD-01~06（categoryId 改造）

### Step 2: 错误码定义
- [x] 创建 `common/src/main/java/.../common/enums/ProductErrorCode.java`
  - STOCK_INSUFFICIENT、PRODUCT_OFF_SHELF、CATEGORY_DEPTH_EXCEEDED、CATEGORY_HAS_PRODUCTS
- 覆盖故事：US-PROD-08、US-PROD-09、US-PROD-07

### Step 3: Category 领域模型
- [x] 创建 `domain/domain-model/src/main/java/.../domain/model/category/CategoryEntity.java`
- 覆盖故事：US-PROD-07

### Step 4: ProductEntity 改造
- [x] 修改 `domain/domain-model/src/main/java/.../domain/model/product/ProductEntity.java`
  - `String category` → `Long categoryId`
- 覆盖故事：US-PROD-01~06

### Step 5: Category 仓储端口
- [x] 创建 `domain/repository-api/src/main/java/.../repository/category/CategoryRepository.java`
- 覆盖故事：US-PROD-07

### Step 6: Product 缓存端口
- [x] 创建 `domain/cache-api/src/main/java/.../cache/product/ProductCacheService.java`
- [x] 创建 `domain/cache-api/src/main/java/.../cache/category/CategoryCacheService.java`
- 覆盖故事：NFR-CACHE-01、NFR-CACHE-02

### Step 7: ProductDomainService 改造与补充
- [x] 修改 `domain/domain-api/src/main/java/.../domain/service/product/ProductDomainService.java`
  - 补充方法：update、delete、updateStatus、deductStock、restoreStock
  - page 方法签名：String category → Long categoryId + List<Long> categoryIds
- [x] 修改 `domain/domain-impl/src/main/java/.../domain/impl/service/product/ProductDomainServiceImpl.java`
  - 实现所有新增方法，库存扣减含乐观锁重试逻辑
- 覆盖故事：US-PROD-01~06、US-PROD-08、US-PROD-09

### Step 8: CategoryDomainService 新增
- [x] 创建 `domain/domain-api/src/main/java/.../domain/service/category/CategoryDomainService.java`
- [x] 创建 `domain/domain-impl/src/main/java/.../domain/impl/service/category/CategoryDomainServiceImpl.java`
- 覆盖故事：US-PROD-07

### Step 9: ProductRepository 改造
- [x] 修改 `domain/repository-api/src/main/java/.../repository/product/ProductRepository.java`
  - page 方法签名改造：支持 categoryIds 列表
  - 新增 countByCategoryId 方法
- 覆盖故事：US-PROD-01、US-PROD-07

### Step 10: Category 仓储实现（MySQL）
- [x] 创建 `infrastructure/repository/mysql-impl/src/main/java/.../repository/mysql/po/category/CategoryPO.java`
- [x] 创建 `infrastructure/repository/mysql-impl/src/main/java/.../repository/mysql/mapper/category/CategoryMapper.java`
- [x] 创建 `infrastructure/repository/mysql-impl/src/main/resources/mapper/category/CategoryMapper.xml`
- [x] 创建 `infrastructure/repository/mysql-impl/src/main/java/.../repository/mysql/impl/category/CategoryRepositoryImpl.java`
- 覆盖故事：US-PROD-07

### Step 11: Product 仓储实现改造
- [x] 修改 `infrastructure/repository/mysql-impl/src/main/java/.../repository/mysql/po/product/ProductPO.java`
  - `String category` → `Long categoryId`
- [x] 修改 `infrastructure/repository/mysql-impl/src/main/java/.../repository/mysql/mapper/product/ProductMapper.java`
  - 新增 countByCategoryId 方法签名
  - 新增 selectTopBySoldCount 方法签名
- [x] 修改 `infrastructure/repository/mysql-impl/src/main/resources/mapper/product/ProductMapper.xml`
  - 分页查询改为 category_id IN 条件
  - 新增 countByCategoryId、selectTopBySoldCount SQL
- [x] 修改 `infrastructure/repository/mysql-impl/src/main/java/.../repository/mysql/impl/product/ProductRepositoryImpl.java`
  - Entity↔PO 转换：category→categoryId
  - 实现新增方法
- 覆盖故事：US-PROD-01~09

### Step 12: 缓存实现（Redis）
- [x] 创建 `infrastructure/cache/redis-impl/src/main/java/.../cache/redis/product/ProductCacheServiceImpl.java`
- [x] 创建 `infrastructure/cache/redis-impl/src/main/java/.../cache/redis/category/CategoryCacheServiceImpl.java`
- 覆盖故事：NFR-CACHE-01、NFR-CACHE-02

### Step 13: Category 应用服务
- [x] 创建 `application/application-api/src/main/java/.../application/api/dto/category/CategoryDTO.java`
- [x] 创建 `application/application-api/src/main/java/.../application/api/dto/category/CategoryTreeNode.java`
- [x] 创建 `application/application-api/src/main/java/.../application/api/dto/category/request/CreateCategoryRequest.java`
- [x] 创建 `application/application-api/src/main/java/.../application/api/dto/category/request/UpdateCategoryRequest.java`
- [x] 创建 `application/application-api/src/main/java/.../application/api/dto/category/request/DeleteCategoryRequest.java`
- [x] 创建 `application/application-api/src/main/java/.../application/api/service/category/CategoryApplicationService.java`
- [x] 创建 `application/application-impl/src/main/java/.../application/impl/service/category/CategoryApplicationServiceImpl.java`
- 覆盖故事：US-PROD-07

### Step 14: Product 应用服务改造
- [x] 修改 `application/application-api/src/main/java/.../application/api/dto/product/ProductDTO.java`
  - `String category` → `Long categoryId` + 新增 `String categoryName`
- [x] 创建 `application/application-api/src/main/java/.../application/api/dto/product/request/UpdateProductRequest.java`
- [x] 创建 `application/application-api/src/main/java/.../application/api/dto/product/request/GetProductRequest.java`
- [x] 创建 `application/application-api/src/main/java/.../application/api/dto/product/request/DeleteProductRequest.java`
- [x] 创建 `application/application-api/src/main/java/.../application/api/dto/product/request/UpdateStatusRequest.java`
- [x] 创建 `application/application-api/src/main/java/.../application/api/dto/product/request/DeductStockRequest.java`
- [x] 创建 `application/application-api/src/main/java/.../application/api/dto/product/request/RestoreStockRequest.java`
- [x] 创建 `application/application-api/src/main/java/.../application/api/dto/product/ProductStatsDTO.java`
- [x] 修改 `application/application-api/src/main/java/.../application/api/dto/product/request/CreateProductRequest.java`
  - `String category` → `Long categoryId`
- [x] 修改 `application/application-api/src/main/java/.../application/api/dto/product/request/ListProductRequest.java`
  - `String category` → `Long categoryId`
- [x] 修改 `application/application-api/src/main/java/.../application/api/service/product/ProductApplicationService.java`
  - 补充方法签名
- [x] 修改 `application/application-impl/src/main/java/.../application/impl/service/product/ProductApplicationServiceImpl.java`
  - 实现所有新增方法，集成缓存
- 覆盖故事：US-PROD-01~09

### Step 15: ProductController 改造
- [x] 修改 `interface/interface-http/src/main/java/.../facade/http/controller/ProductController.java`
  - 补充端点：detail、admin/list、admin/update、admin/delete、admin/update-status、stats
  - 调整现有端点路径
- 覆盖故事：US-PROD-01~06

### Step 16: CategoryController 新增
- [x] 创建 `interface/interface-http/src/main/java/.../facade/http/controller/CategoryController.java`
  - 端点：public/category/tree、admin/category/create、admin/category/update、admin/category/delete
- 覆盖故事：US-PROD-07

### Step 17: InternalProductController 新增
- [x] 创建 `interface/interface-http/src/main/java/.../facade/http/controller/InternalProductController.java`
  - 端点：internal/product/get、internal/product/deduct-stock、internal/product/restore-stock
- [x] 创建内部接口请求/响应 DTO（在 application-api 中）
- 覆盖故事：US-PROD-08、US-PROD-09

### Step 18: 代码生成摘要文档
- [x] 创建 `aidlc-docs/construction/unit-product/code/code-generation-summary.md`

---

## 故事覆盖追踪

| 故事 | 覆盖步骤 | 状态 |
|------|---------|------|
| US-PROD-01 浏览商品列表 | Step 4,7,9,11,14,15 | [x] |
| US-PROD-02 查看商品详情 | Step 4,7,14,15 | [x] |
| US-PROD-03 创建商品 | Step 4,7,14,15 | [x] |
| US-PROD-04 编辑商品 | Step 4,7,14,15 | [x] |
| US-PROD-05 删除商品 | Step 7,14,15 | [x] |
| US-PROD-06 商品上架/下架 | Step 7,14,15 | [x] |
| US-PROD-07 管理商品分类 | Step 1,2,3,5,8,10,13,16 | [x] |
| US-PROD-08 库存扣减 | Step 2,7,14,17 | [x] |
| US-PROD-09 库存恢复 | Step 2,7,14,17 | [x] |
