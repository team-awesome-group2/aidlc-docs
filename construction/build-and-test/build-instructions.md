# 构建指令 — unit-product (Product Service)

## 前置条件

- Java 21
- Maven 3.8+
- MySQL 8.4（本地端口 3307，数据库 `awsome_shop_product`）
- Redis（本地端口 6379）

## 构建步骤

### 1. 进入项目目录

```bash
cd awsome-shop-product-service
```

### 2. 构建项目（跳过测试）

```bash
mvn clean package -DskipTests
```

### 3. 验证构建成功

- 预期输出：`BUILD SUCCESS`
- 构建产物：`bootstrap/target/awsome-shop-product-service-1.0.0-SNAPSHOT.jar`

### 4. 数据库迁移

确保 MySQL 已启动，Flyway 会在应用启动时自动执行迁移：
- V1: 创建 test 表
- V2: 创建 product 表
- V3: 创建 category 表 + 修改 product 表（category → category_id）

### 5. 启动应用

```bash
mkdir -p /tmp/awsome-shop/product
nohup java -jar bootstrap/target/awsome-shop-product-service-1.0.0-SNAPSHOT.jar \
  --spring.profiles.active=local \
  > /tmp/awsome-shop/product/app.log 2>&1 &
```

### 6. 健康检查

```bash
curl http://localhost:8002/actuator/health
```

### 7. 停止应用

```bash
lsof -ti:8002 | xargs kill -9
```

## 常见问题

### Flyway 迁移失败
- 确认 MySQL 已启动且端口 3307 可访问
- 确认数据库 `awsome_shop_product` 已创建
- 如果 V3 迁移失败（category 列已不存在），可能需要清理 flyway_schema_history 表

### Redis 连接失败
- 确认 Redis 已启动且端口 6379 可访问
- 缓存连接失败不会阻止应用启动，但缓存功能不可用
