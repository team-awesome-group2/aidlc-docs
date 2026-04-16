# 功能设计计划 — 单元 2: unit-product（商品服务）

## 计划概览

基于 INCEPTION 阶段产物（需求文档、用户故事 US-PROD-01~09、应用设计、内部接口契约）和组长的 category 字段改造分析，为 Product Service 生成详细的功能设计。

## 执行步骤

- [x] 步骤 1：定义领域实体（ProductEntity 改造 + CategoryEntity 新增）
- [x] 步骤 2：定义业务逻辑模型（Product 补充方法 + Category 全套 + 统计接口）
- [x] 步骤 3：定义业务规则（校验规则、状态流转、库存并发控制、分类约束）
- [x] 步骤 4：定义可测试属性（PBT-01 合规）
- [x] 步骤 5：生成功能设计产物文件

## 设计变更说明（来自组长分析）

- `product` 表 `category VARCHAR(100)` → `category_id BIGINT`（关联 category 表）
- 数据库为空库，Flyway 迁移可直接修改列定义，无需数据迁移
- 影响 Product Service 内部 10 个 Java 文件

---

# 功能设计问题

请回答以下问题，在 `[Answer]:` 后填写字母选项。如果没有匹配的选项，选择最后一个"Other"并描述。

## Question 1
分类树的最大层级深度是多少？

A) 2 级（顶级分类 → 子分类）
B) 3 级（顶级 → 二级 → 三级）
C) 不限层级（递归树，任意深度）
D) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 2
删除分类时，如果该分类有子分类（但子分类下没有商品），应该如何处理？

A) 拒绝删除（必须先删除所有子分类）
B) 级联删除（同时删除所有子分类）
C) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 3
商品的 `categoryId` 是否允许为空（即商品可以不属于任何分类）？

A) 不允许为空，创建商品时必须指定分类
B) 允许为空，商品可以暂时不归类
C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 4
商品更新时，哪些字段不允许修改？

A) 仅 SKU 不可修改（创建后不可变）
B) SKU 和积分价格都不可修改
C) 所有字段都可以修改
D) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 5
库存扣减失败（乐观锁冲突）时的重试策略是什么？

A) 不重试，直接返回失败（由调用方 Order Service 处理）
B) Product Service 内部自动重试 1~3 次
C) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 6
商品统计接口（`POST /api/v1/product/stats`）中的"热门商品"排行依据是什么？

A) 按 `soldCount`（已兑换数量）降序排列
B) 按最近 30 天的兑换量降序排列
C) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 7
已下架的商品是否允许执行库存扣减（内部接口调用）？

A) 不允许，下架商品拒绝扣减（返回 PRODUCT_OFF_SHELF 错误）
B) 允许，内部接口不检查上下架状态（由 Order Service 在兑换前校验）
C) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 8
商品列表查询（员工端）是否需要按分类的树形结构筛选（即选择父分类时自动包含所有子分类下的商品）？

A) 是，选择父分类时包含所有子分类的商品
B) 否，只精确匹配所选分类 ID 的商品
C) Other (please describe after [Answer]: tag below)

[Answer]: A

