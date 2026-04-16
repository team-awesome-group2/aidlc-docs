# Awsome Shop 积分商城 — 项目开发总结

## 项目概况

- 项目类型：棕地项目（6 个微服务 + 1 个前端）二次开发
- 本人负责：Unit 2 — Product Service（商品服务）
- 开发周期：2026-04-15 ~ 2026-04-16
- 技术栈：Java 21 / Spring Boot 3.4.1 / React 19 / TypeScript

---

## 一、构思阶段（Inception）

### 做得好的地方

1. **完整的逆向工程**：对现有 6 个微服务进行了全面分析，生成了 8 个产物（业务概览、架构文档、API 文档、组件清单等），为后续开发提供了清晰的上下文
2. **需求分析有深度**：11 个澄清问题覆盖了认证模型、跨服务一致性、分类管理、测试策略等关键决策点，避免了开发中的返工
3. **用户故事粒度合理**：41 个用户故事按 3 个角色（员工、管理员、系统）划分，P0/P1 优先级清晰
4. **单元拆分科学**：按微服务边界拆分为 6 个工作单元，依赖关系明确（Auth → Product+Points → Order → Gateway → Frontend），支持并行开发
5. **内部接口契约前置**：在编码前定义了 10 个跨服务接口的完整 HTTP 契约，减少了联调时的沟通成本

### 遇到的问题

1. **组长的设计变更**：开发中途收到组长关于 category 字段改造的分析（VARCHAR → category_id 外键），需要调整已完成的功能设计
2. **需求答案不完整**：第一轮需求问题只回答了 2/11，需要额外一轮澄清才能继续

### 改进建议

1. **接口契约应更早对齐**：字段命名规范（如 `id` vs `productId`）应在契约阶段就统一，而不是联调时才发现
2. **设计变更流程**：中途的设计变更应有正式的变更记录和影响分析，而不是口头传达

---

## 二、构建阶段（Construction）

### 做得好的地方

1. **DDD + 六边形架构一致性**：严格遵循现有项目的分层架构（domain → application → interface → bootstrap），新增代码与原有风格统一
2. **Flyway 迁移管理**：数据库变更通过版本化迁移脚本管理（V3__create_category_and_modify_product），可重复执行
3. **功能设计先行**：在编码前完成了业务规则文档、领域实体设计、业务逻辑模型，代码生成有据可依
4. **NFR 需求明确**：Redis 缓存策略（商品 30min TTL、分类树 1h TTL）、API 响应时间目标（查询 <500ms）等非功能需求在编码前确定

### 遇到的问题

1. **WebFlux/WebMvc 冲突（严重）**：代码生成时错误引入了 `spring-boot-starter-webflux`，导致所有 Controller 返回 404。根因是 Product Service 作为被调用方不需要 WebClient，但代码生成模板未区分调用方/被调用方
2. **前后端字段名不一致**：后端 `GetProductRequest` 用 `id`，前端用 `productId`，导致商品详情页报"商品不存在"。3 个 Request DTO + 2 个 Controller 需要修改
3. **Order Service 同样的 WebFlux 问题**：其他组员的 Order Service 也遇到了相同的 WebFlux 依赖问题，说明这是代码生成模板的系统性缺陷

### 改进建议

1. **代码生成模板审查**：WebFlux starter 不应出现在非 reactive 项目中。代码生成规则应区分"需要 WebClient 的调用方"和"纯 WebMvc 的被调用方"
2. **字段命名规范强制化**：在内部接口契约中明确字段命名规范（如统一使用 `{entity}Id` 格式），并在代码生成时自动对齐
3. **编译验证应前置**：代码生成完成后应立即执行 `mvn compile` 验证，而不是等到联调阶段才发现编译问题

---

## 三、部署与测试阶段（Operations）

### 做得好的地方

1. **完整的本地环境搭建**：从零安装 MySQL（port 3307）和 Redis（port 6379），配置与 application-local.yml 完全匹配
2. **全栈启动脚本**：实现了带健康检查的服务启动流程，按依赖顺序启动（Auth → Product+Points → Order → Gateway → Frontend）
3. **Playwright E2E 测试覆盖全面**：25 个测试用例覆盖 9 个功能模块，包括正向流程和异常场景（重复注册、错误密码），核心交易链路（兑换 → 扣积分 → 减库存 → 创建订单 → 取消 → 退款）全部验证
4. **API 级数据完整性验证**：不仅验证 UI 交互，还通过 API 断言验证积分余额、库存数量等数据一致性

### 遇到的问题

1. **服务启动 hang 住**：首次启动测试时使用 `sleep 15 && tail` 的方式，如果应用启动卡住会导致整个命令 hang。后改为轮询健康检查 + 超时机制
2. **数据库名不一致**：Points Service 的数据库名是 `awsome_shop_point`（无 s），而我创建的是 `awsome_shop_points`，导致启动失败
3. **进程管理混乱**：`pkill -f "awsome-shop"` 会误杀 vite 前端进程（因为工作目录包含 awsome-shop），后改为 `pkill -f "java.*jar"`
4. **服务启动慢**：5 个 Java 服务同时启动，在资源有限的环境下需要 60-70 秒，Flyway 迁移 + Druid 连接池初始化是主要耗时
5. **取消订单退款未同步**：Order Service 取消订单后积分和库存未立即恢复，可能是异步补偿机制（CompensationScheduler）的执行延迟

### 改进建议

1. **Docker Compose 统一环境**：应提供 docker-compose.yml 一键启动所有基础设施（MySQL、Redis）和服务，避免手动配置端口、数据库名等
2. **数据库命名规范**：所有服务的数据库名应在项目文档中统一列出，避免猜测
3. **测试数据管理**：E2E 测试应有独立的数据初始化脚本，而不是依赖 Flyway seed data + 测试中动态创建
4. **CI/CD 集成**：Playwright 测试应集成到 CI 流水线，每次提交自动运行

---

## 关键数据

| 指标 | 数值 |
|------|------|
| Product Service 修改文件数 | 6 |
| 修复 Bug 数 | 2 (WebFlux 冲突 + 字段名不一致) |
| 提交给其他组员的 Bug Report | 2 (前端分类选择器 + Order 退款) |
| E2E 测试用例数 | 25 |
| E2E 测试通过率 | 100% |
| 全栈启动时间 | ~70s |
| E2E 测试执行时间 | 47.8s |
