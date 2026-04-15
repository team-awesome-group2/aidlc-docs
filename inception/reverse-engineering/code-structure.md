# Code Structure

## Build System

- **Type**: Maven (multi-module)
- **Configuration**: 每个后端服务为独立 Maven 项目，包含约 26 个子模块
- **Frontend**: npm + Vite 7

## DDD 六边形架构分层 (所有后端服务统一)

```
service-root/
├── common/                          # 公共基础层
│   └── src/main/java/.../common/
│       ├── annotation/              # 自定义注解 (RequireOwnerPermission)
│       ├── dto/                     # 通用 DTO (PageResult)
│       ├── enums/                   # 错误码枚举 (ErrorCode, SystemErrorCode, ParamErrorCode, SampleErrorCode)
│       ├── exception/               # 异常体系 (BaseException, BusinessException, SystemException, ParameterException)
│       └── result/                  # 统一返回 (Result)
│
├── domain/
│   ├── domain-model/                # 领域实体 (纯 POJO)
│   ├── domain-api/                  # 领域服务接口
│   ├── domain-impl/                 # 领域服务实现
│   ├── repository-api/              # 仓储端口 (Port)
│   ├── cache-api/                   # 缓存端口 (Port)
│   ├── mq-api/                      # 消息队列端口 (Port)
│   └── security-api/                # 安全端口 (Port - EncryptionService)
│
├── infrastructure/
│   ├── repository/mysql-impl/       # MySQL 仓储适配器 (MyBatis-Plus)
│   ├── cache/redis-impl/            # Redis 缓存适配器
│   ├── mq/sqs-impl/                 # SQS 消息队列适配器 (空)
│   ├── security/jwt-impl/           # JWT + AES 安全适配器
│   └── gateway/gateway-impl/        # Gateway 专有 (仅 gateway-service)
│
├── application/
│   ├── application-api/             # 应用服务接口 + DTO + Request
│   └── application-impl/           # 应用服务实现
│
├── interface/
│   ├── interface-http/              # HTTP 控制器 (REST API)
│   └── interface-consumer/          # 消息消费者 (空)
│
└── bootstrap/                       # Spring Boot 启动模块
    └── src/main/resources/
        ├── application.yml          # 通用配置
        ├── application-local.yml    # 本地开发配置
        └── db/migration/            # Flyway SQL 迁移脚本
```

## Existing Files Inventory

### Auth Service (awsome-shop-auth-service)

- `domain/domain-model/.../test/TestEntity.java` — 测试占位实体
- `domain/domain-api/.../test/TestDomainService.java` — 测试领域服务接口
- `domain/domain-impl/.../test/TestDomainServiceImpl.java` — 测试领域服务实现
- `domain/repository-api/.../test/TestRepository.java` — 测试仓储接口
- `domain/security-api/.../EncryptionService.java` — 加密服务接口
- `application/application-api/.../test/` — 测试应用服务接口 + DTO
- `application/application-impl/.../test/` — 测试应用服务实现
- `interface/interface-http/.../test/TestController.java` — 测试控制器
- `infrastructure/security/jwt-impl/.../AesEncryptionServiceImpl.java` — AES 加密实现
- `bootstrap/src/main/resources/db/migration/V1__create_test_table.sql` — 测试表

### Product Service (awsome-shop-product-service)

- `domain/domain-model/.../product/ProductEntity.java` — 商品领域实体 ✅
- `domain/domain-model/.../test/TestEntity.java` — 测试占位实体
- `domain/domain-api/.../product/ProductDomainService.java` — 商品领域服务接口 ✅
- `domain/domain-impl/.../product/ProductDomainServiceImpl.java` — 商品领域服务实现 ✅
- `domain/repository-api/.../product/ProductRepository.java` — 商品仓储接口 ✅
- `application/application-api/.../product/` — 商品应用服务 + DTO ✅
- `application/application-impl/.../product/` — 商品应用服务实现 ✅
- `interface/interface-http/.../product/ProductController.java` — 商品控制器 ✅
- `infrastructure/repository/mysql-impl/.../product/` — 商品 MySQL 实现 ✅
- `bootstrap/src/main/resources/db/migration/V2__create_product_table.sql` — 商品表 ✅

### Points Service (awsome-shop-points-service)

- 仅包含 test 占位代码，无积分业务实现

### Order Service (awsome-shop-order-service)

- 仅包含 test 占位代码，无订单业务实现

### Gateway Service (awsome-shop-gateway-service)

- `domain/domain-model/.../auth/AuthenticationResult.java` — 认证结果值对象 ✅
- `domain/domain-model/.../auth/TokenInfo.java` — Token 信息值对象 ✅
- `domain/domain-api/.../auth/AuthenticationService.java` — 认证服务接口 ✅
- `infrastructure/gateway/gateway-impl/.../filter/AccessLogFilter.java` — 访问日志过滤器 ✅
- `infrastructure/gateway/gateway-impl/.../filter/AuthenticationGatewayFilter.java` — 认证过滤器 ✅
- `infrastructure/gateway/gateway-impl/.../filter/OperatorIdInjectionFilter.java` — 操作员 ID 注入 ✅
- `infrastructure/gateway/gateway-impl/.../auth/client/AuthServiceClient.java` — Auth 服务客户端 ✅
- `infrastructure/gateway/gateway-impl/.../swagger/SwaggerServersRewriteGatewayFilterFactory.java` — Swagger 重写 ✅

### Frontend (awsome-shop-frontend)

- `src/pages/Login/index.tsx` — 登录页 ✅
- `src/pages/ShopHome/index.tsx` — 员工商城首页 ✅ (Mock 数据)
- `src/pages/Dashboard/index.tsx` — 管理员仪表盘 ✅ (Mock 数据)
- `src/components/Layout/EmployeeLayout.tsx` — 员工布局 ✅
- `src/components/Layout/AdminLayout.tsx` — 管理员布局 ✅
- `src/components/AvatarMenu.tsx` — 头像菜单 ✅
- `src/router/index.tsx` — 路由配置 ✅
- `src/router/AuthGuard.tsx` — 路由守卫 ✅
- `src/store/useAuthStore.ts` — 认证状态 ✅ (Mock 登录)
- `src/store/useAppStore.ts` — 应用状态 ✅
- `src/services/request.ts` — Axios HTTP 客户端 ✅
- `src/i18n/` — 国际化配置 ✅
- `src/theme/index.ts` — MUI 主题 ✅

## Design Patterns

### Hexagonal Architecture (Ports & Adapters)

- **Location**: 所有后端服务
- **Purpose**: 领域逻辑与基础设施解耦
- **Implementation**: domain-api 定义 Port 接口，infrastructure 提供 Adapter 实现

### Repository Pattern

- **Location**: domain/repository-api + infrastructure/repository/mysql-impl
- **Purpose**: 数据访问抽象
- **Implementation**: 领域层定义 Repository 接口，MySQL 实现层通过 MyBatis-Plus 实现

### Application Service Pattern

- **Location**: application/application-api + application/application-impl
- **Purpose**: 编排领域服务，处理 DTO 转换
- **Implementation**: 应用服务接口定义用例，实现层协调领域服务

### Global Filter Chain (Gateway)

- **Location**: gateway-service/infrastructure/gateway
- **Purpose**: 请求处理管道
- **Implementation**: AccessLog → Authentication → OperatorIdInjection，有序过滤器链

### Zustand Store Pattern (Frontend)

- **Location**: frontend/src/store
- **Purpose**: 轻量级状态管理
- **Implementation**: Zustand + persist 中间件，持久化到 localStorage

## Critical Dependencies

### Spring Boot 3.4.1 / 3.5.10

- **Version**: 3.4.1 (业务服务) / 3.5.10 (Gateway)
- **Usage**: 应用框架核心
- **Purpose**: 自动配置、依赖注入、Web 框架

### MyBatis-Plus 3.5.7

- **Version**: 3.5.7
- **Usage**: 所有后端服务的 ORM
- **Purpose**: 简化 MyBatis 开发，提供 CRUD 封装、分页、逻辑删除、乐观锁

### Spring Cloud Gateway

- **Version**: Spring Cloud 2025.0.0
- **Usage**: Gateway 服务
- **Purpose**: 响应式 API 网关，路由、过滤器

### React 19 + MUI 6

- **Version**: React 19.2 / MUI 6.5
- **Usage**: 前端 UI
- **Purpose**: 组件化 UI 开发

### JJWT 0.12.6

- **Version**: 0.12.6
- **Usage**: Auth 服务 + Gateway
- **Purpose**: JWT Token 生成与验证
