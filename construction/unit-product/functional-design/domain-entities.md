# 领域实体 — unit-product

## 1. ProductEntity（改造）

### 字段定义

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | Long | 自增主键 | 商品 ID |
| name | String | 是 | 商品名称，最长 200 字符 |
| sku | String | 是 | 商品编号，唯一，创建后不可修改，最长 100 字符 |
| categoryId | Long | 是 | 分类 ID，关联 category 表（原 category VARCHAR 改造） |
| brand | String | 否 | 品牌，最长 100 字符 |
| pointsPrice | Integer | 是 | 积分价格，必须 > 0 |
| marketPrice | BigDecimal | 否 | 市场参考价 |
| stock | Integer | 是 | 当前库存，>= 0 |
| soldCount | Integer | 是 | 已兑换数量，默认 0 |
| status | Integer | 是 | 状态：0=已下架，1=已上架，默认 0 |
| description | String | 否 | 商品描述 |
| imageUrl | String | 否 | 主图 URL，最长 500 字符 |
| subtitle | String | 否 | 副标题/卖点，最长 500 字符 |
| deliveryMethod | String | 否 | 配送方式，最长 200 字符 |
| serviceGuarantee | String | 否 | 服务保障，最长 500 字符 |
| promotion | String | 否 | 促销活动，最长 200 字符 |
| colors | String | 否 | 可选颜色（逗号分隔），最长 500 字符 |
| specs | List&lt;Map&lt;String,String&gt;&gt; | 否 | 规格参数（JSON） |
| createdAt | LocalDateTime | 自动 | 创建时间 |
| updatedAt | LocalDateTime | 自动 | 更新时间 |

### 持久化层附加字段（PO 层）

| 字段 | 类型 | 说明 |
|------|------|------|
| createdBy | Long | 创建人 |
| updatedBy | Long | 更新人 |
| deleted | Integer | 逻辑删除：0=未删除，1=已删除 |
| version | Integer | 乐观锁版本号 |

### 变更说明

- `category String` → `categoryId Long`：关联 category 表主键
- 新增 Flyway 迁移脚本处理列变更

---

## 2. CategoryEntity（新增）

### 字段定义

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | Long | 自增主键 | 分类 ID |
| name | String | 是 | 分类名称，最长 100 字符 |
| parentId | Long | 否 | 父分类 ID，NULL 表示顶级分类 |
| sortOrder | Integer | 是 | 排序序号，默认 0，值越小越靠前 |
| status | Integer | 是 | 状态：0=禁用，1=启用，默认 1 |
| createdAt | LocalDateTime | 自动 | 创建时间 |
| updatedAt | LocalDateTime | 自动 | 更新时间 |

### 持久化层附加字段（PO 层）

| 字段 | 类型 | 说明 |
|------|------|------|
| deleted | Integer | 逻辑删除：0=未删除，1=已删除 |

### 树形结构约束

- 最大深度：3 级（顶级 → 二级 → 三级）
- 顶级分类：parentId = NULL
- 二级分类：parentId 指向顶级分类
- 三级分类：parentId 指向二级分类
- 不允许创建第 4 级分类

---

## 3. 实体关系

```
CategoryEntity (1) ──── (N) ProductEntity
    parentId ──── CategoryEntity (自引用，树形)
```

文本替代：
- 一个 Category 可以关联多个 Product（通过 product.categoryId）
- Category 通过 parentId 自引用形成树形结构（最多 3 级）
- Product 必须关联一个 Category（categoryId NOT NULL）

---

## 4. 数据库迁移（Flyway）

### V3__create_category_table_and_modify_product.sql

```sql
-- 1. 创建分类表
CREATE TABLE `category` (
    `id`         BIGINT       NOT NULL AUTO_INCREMENT COMMENT '分类ID',
    `name`       VARCHAR(100) NOT NULL COMMENT '分类名称',
    `parent_id`  BIGINT       DEFAULT NULL COMMENT '父分类ID，NULL表示顶级分类',
    `sort_order` INT          NOT NULL DEFAULT 0 COMMENT '排序序号',
    `status`     TINYINT      NOT NULL DEFAULT 1 COMMENT '状态 0-禁用 1-启用',
    `created_at` DATETIME     NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    `updated_at` DATETIME     NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
    `deleted`    TINYINT      NOT NULL DEFAULT 0 COMMENT '逻辑删除 0-未删除 1-已删除',
    PRIMARY KEY (`id`),
    INDEX `idx_parent_id` (`parent_id`)
) ENGINE = InnoDB DEFAULT CHARSET = utf8mb4 COLLATE = utf8mb4_unicode_ci COMMENT = '商品分类表';

-- 2. 修改 product 表：category VARCHAR → category_id BIGINT
ALTER TABLE `product` ADD COLUMN `category_id` BIGINT DEFAULT NULL COMMENT '分类ID' AFTER `category`;
ALTER TABLE `product` DROP INDEX `idx_category`;
ALTER TABLE `product` DROP COLUMN `category`;
CREATE INDEX `idx_category_id` ON `product`(`category_id`);
```
