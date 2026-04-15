# Component Inventory

## Application Packages

| Package | Purpose | Port | Implementation Status |
|---------|---------|------|----------------------|
| awsome-shop-auth-service | 用户认证与授权 | 8001 | 🟡 脚手架 (仅 Test CRUD) |
| awsome-shop-product-service | 商品目录管理 | 8002 | 🟢 部分实现 (Create + List) |
| awsome-shop-points-service | 积分管理 | 8003 | 🟡 脚手架 (仅 Test CRUD) |
| awsome-shop-order-service | 订单管理 | 8004 | 🟡 脚手架 (仅 Test CRUD) |
| awsome-shop-gateway-service | API 网关 | 8080 | 🟢 完整实现 |
| awsome-shop-frontend | 前端 SPA | 5173 | 🟢 UI 框架完成 (Mock 数据) |

## Infrastructure Packages (per service)

每个后端服务包含以下基础设施模块:

| Module | Type | Purpose | Status |
|--------|------|---------|--------|
| infrastructure/repository/mysql-impl | MySQL Adapter | 数据持久化 | ✅ 已实现 |
| infrastructure/cache/redis-impl | Redis Adapter | 缓存 | ✅ 已配置 |
| infrastructure/mq/sqs-impl | SQS Adapter | 消息队列 | ⬜ 空模块 |
| infrastructure/security/jwt-impl | JWT + AES Adapter | 安全 | ✅ 已实现 |
| infrastructure/gateway/gateway-impl | Gateway Filters | 网关过滤器 (仅 Gateway) | ✅ 已实现 |

## Shared Packages (per service)

| Module | Type | Purpose |
|--------|------|---------|
| common | Utilities | 异常体系、错误码、Result 包装、PageResult、注解 |
| domain/domain-model | Models | 领域实体 (纯 POJO) |
| domain/domain-api | Interfaces | 领域服务接口 |
| domain/repository-api | Port | 仓储端口接口 |
| domain/cache-api | Port | 缓存端口接口 |
| domain/mq-api | Port | 消息队列端口接口 |
| domain/security-api | Port | 安全端口接口 (EncryptionService) |

## Test Packages

| Package | Type | Purpose |
|---------|------|---------|
| bootstrap/src/test | Unit/Integration | Spring Boot 测试 (基础配置) |

## Frontend Modules

| Module | Purpose | Status |
|--------|---------|--------|
| src/pages/Login | 登录页 | ✅ 完成 (Mock) |
| src/pages/ShopHome | 员工商城首页 | ✅ 完成 (Mock 数据) |
| src/pages/Dashboard | 管理员仪表盘 | ✅ 完成 (Mock 数据) |
| src/components/Layout | 布局组件 | ✅ 完成 |
| src/router | 路由 + 鉴权 | ✅ 完成 |
| src/store | 状态管理 | ✅ 完成 (Mock) |
| src/services | HTTP 客户端 | ✅ 配置完成 |
| src/i18n | 国际化 | ✅ 完成 (zh/en) |
| src/theme | 主题 | ✅ 完成 (亮/暗) |

## Total Count

- **Total Repositories**: 6
- **Backend Services**: 5 (4 business + 1 gateway)
- **Frontend**: 1
- **Maven Modules per Service**: ~26
- **Total Maven Modules**: ~130
- **Implementation Completeness**: ~25% (架构完整，业务逻辑大部分待实现)
