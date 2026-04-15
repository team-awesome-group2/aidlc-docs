# Technology Stack

## Programming Languages

| Language | Version | Usage |
|----------|---------|-------|
| Java | 21 | 后端微服务 (auth, product, points, order, gateway) |
| TypeScript | 5.9 | 前端 SPA |
| SQL | - | 数据库迁移脚本 (Flyway) |

## Frameworks

| Framework | Version | Purpose |
|-----------|---------|---------|
| Spring Boot | 3.4.1 | 业务微服务应用框架 |
| Spring Boot | 3.5.10 | Gateway 服务应用框架 |
| Spring Cloud | 2025.0.0 | 微服务基础设施 |
| Spring Cloud Gateway | - | 响应式 API 网关 |
| Spring WebFlux | - | Gateway 响应式 Web 框架 |
| React | 19.2 | 前端 UI 框架 |
| MUI (Material-UI) | 6.5 | 前端组件库 |

## ORM & Data Access

| Tool | Version | Purpose |
|------|---------|---------|
| MyBatis-Plus | 3.5.7 | ORM 框架 (CRUD 封装、分页、逻辑删除、乐观锁) |
| Druid | 1.2.20 | 数据库连接池 |
| Flyway | - | 数据库迁移管理 |

## Infrastructure

| Service | Version | Purpose |
|---------|---------|---------|
| MySQL | 8.4.8 | 关系型数据库 (每服务独立库) |
| Redis | - | 缓存 (Lettuce 客户端) |
| AWS SQS | SDK 2.20.0 | 消息队列 (已定义接口，未实现) |

## Security

| Tool | Version | Purpose |
|------|---------|---------|
| JJWT | 0.12.6 | JWT Token 生成与验证 |
| AES-256 | - | 敏感数据加密 |
| BCrypt | - | 密码哈希 (Spring Security) |

## Build Tools

| Tool | Version | Purpose |
|------|---------|---------|
| Maven | 3.x | Java 项目构建 |
| Vite | 7.3 | 前端构建工具 |
| npm | - | 前端包管理 |

## API Documentation

| Tool | Version | Purpose |
|------|---------|---------|
| SpringDoc OpenAPI | 2.7.0 | API 文档生成 |
| Swagger UI | - | API 文档可视化 (Gateway 聚合) |

## Observability

| Tool | Version | Purpose |
|------|---------|---------|
| Micrometer Tracing | 1.3.5 | 分布式追踪 |
| Prometheus | - | 指标暴露 (/actuator/prometheus) |
| Logstash Logback Encoder | 7.4 | 结构化日志 |

## Frontend Libraries

| Library | Version | Purpose |
|---------|---------|---------|
| Zustand | 5.0 | 状态管理 |
| React Router | 7.13 | 路由管理 |
| Axios | 1.13 | HTTP 客户端 |
| i18next | 25.8 | 国际化 |
| Emotion | 11.14 | CSS-in-JS (MUI 依赖) |

## Testing Tools

| Tool | Version | Purpose |
|------|---------|---------|
| JUnit 5 | - | Java 单元测试 (Spring Boot Starter Test) |
| ESLint | 9.39 | 前端代码检查 |
