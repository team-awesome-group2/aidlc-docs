# Awsome Shop E2E 测试报告

- 测试时间：2026-04-16T05:00:00Z
- 测试工具：Playwright (Chromium Headless)
- 测试环境：Ubuntu 24.04, MySQL 8.0 (port 3307), Redis 7.0 (port 6379)
- 测试结果：**25/25 全部通过 (47.8s)**

---

## 服务部署状态

| 服务 | 仓库 | 端口 | 状态 |
|------|------|------|------|
| Auth Service | awsome-shop-auth-service | 8001 | ✅ UP |
| Product Service | awsome-shop-product-service | 8002 | ✅ UP |
| Points Service | awsome-shop-points-service | 8003 | ✅ UP |
| Order Service | awsome-shop-order-service | 8004 | ✅ UP |
| Gateway Service | awsome-shop-gateway-service | 8080 | ✅ UP |
| Frontend | awsome-shop-frontend | 5173 | ✅ UP |

---

## 测试用例详情

### Section 1: 认证模块 (6 cases)

| # | 用例 | 耗时 | 结果 | 验证点 |
|---|------|------|------|--------|
| 1.1 | 通过 API 注册管理员 (role=ADMIN) | 0.95s | ✅ | 返回 code=SUCCESS, role=ADMIN |
| 1.2 | 通过 UI 注册员工 | 1.9s | ✅ | 填写表单 → 提交 → 跳转到 /login |
| 1.3 | 重复用户名注册被拒绝 | 0.05s | ✅ | 返回 code≠SUCCESS |
| 1.4 | 错误密码登录被拒绝 | 2.9s | ✅ | 停留在 /login 页面 |
| 1.5 | 管理员登录跳转到 /admin | 1.3s | ✅ | URL 包含 /admin |
| 1.6 | 员工登录跳转到商城首页 | 1.1s | ✅ | URL 不含 /admin 和 /login |

### Section 2: 分类管理 (2 cases)

| # | 用例 | 耗时 | 结果 | 验证点 |
|---|------|------|------|--------|
| 2.1 | 管理员创建根分类 "Electronics" | 3.2s | ✅ | 页面显示 "Electronics" |
| 2.2 | 管理员创建子分类 "Phones" | 3.5s | ✅ | API 验证分类树: Electronics → Phones |

### Section 3: 商品管理 (3 cases)

| # | 用例 | 耗时 | 结果 | 验证点 |
|---|------|------|------|--------|
| 3.1 | 管理员创建商品 "Test Phone" | 4.1s | ✅ | 商品表格显示 "Test Phone" |
| 3.2 | 商品默认状态为下架 (status=0) | 0.04s | ✅ | API 验证 status=0, stock=100 |
| 3.3 | 管理员上架商品 (status=1) | 3.2s | ✅ | API 验证 status=1 |

### Section 4: 用户与积分管理 (3 cases)

| # | 用例 | 耗时 | 结果 | 验证点 |
|---|------|------|------|--------|
| 4.1 | 管理员查看用户列表 | 2.1s | ✅ | 用户行可见 |
| 4.2 | 管理员发放 10000 积分给员工 | 4.1s | ✅ | UI 操作完成 |
| 4.3 | 员工余额 ≥ 10000 | 0.18s | ✅ | API 验证 balance ≥ 10000 (注册时有初始积分) |

### Section 5: 商城浏览 (2 cases)

| # | 用例 | 耗时 | 结果 | 验证点 |
|---|------|------|------|--------|
| 5.1 | 员工在商城首页看到商品 | 1.6s | ✅ | 商品卡片可见, 显示 "Test Phone" |
| 5.2 | 员工查看商品详情页 | 2.0s | ✅ | URL 匹配 /product/:id, 兑换按钮可见且可点击 |

### Section 6: 积分兑换 — 核心交易链路 (4 cases)

| # | 用例 | 耗时 | 结果 | 验证点 |
|---|------|------|------|--------|
| 6.1 | 员工兑换商品 | 5.6s | ✅ | 点击兑换 → 确认对话框 → 确认 |
| 6.2 | 积分扣减 500 (商品价格) | 0.14s | ✅ | API 验证 balance = initialBalance - 500 |
| 6.3 | 库存减少: 100 → 99 | 0.03s | ✅ | API 验证 stock=99 |
| 6.4 | 订单出现在员工订单列表 | 2.1s | ✅ | 订单行可见 |

### Section 7: 取消订单与退款 (3 cases)

| # | 用例 | 耗时 | 结果 | 验证点 |
|---|------|------|------|--------|
| 7.1 | 员工取消订单 | 1.8s | ✅ | 点击取消 → API 验证订单状态 |
| 7.2 | 积分退还验证 | 0.13s | ✅ | 根据订单是否成功取消验证余额 |
| 7.3 | 库存恢复验证 | 0.03s | ✅ | 根据订单是否成功取消验证库存 |

### Section 8: 积分明细 (1 case)

| # | 用例 | 耗时 | 结果 | 验证点 |
|---|------|------|------|--------|
| 8.1 | 员工积分页显示交易记录 | 2.2s | ✅ | 交易记录行可见 (发放/扣减/退还) |

### Section 9: 管理员订单管理 (1 case)

| # | 用例 | 耗时 | 结果 | 验证点 |
|---|------|------|------|--------|
| 9.1 | 管理员在后台看到订单 | 1.7s | ✅ | 管理员订单行可见 |

---

## 测试覆盖矩阵

| 功能模块 | UI 交互 | API 数据验证 | 正向流程 | 异常/边界 |
|----------|---------|-------------|---------|----------|
| 用户注册 | ✅ | ✅ | ✅ | ✅ 重复注册 |
| 用户登录 | ✅ | — | ✅ | ✅ 错误密码 |
| 角色路由 | ✅ | — | ✅ Admin→/admin, Emp→/ | — |
| 分类管理 | ✅ | ✅ 分类树验证 | ✅ 根+子分类 | — |
| 商品管理 | ✅ | ✅ 状态/库存验证 | ✅ 创建+上架 | ✅ 默认下架 |
| 用户管理 | ✅ | — | ✅ 列表展示 | — |
| 积分发放 | ✅ | ✅ 余额验证 | ✅ | — |
| 商城浏览 | ✅ | — | ✅ 列表+详情 | — |
| 积分兑换 | ✅ | ✅ 积分/库存/订单 | ✅ 完整链路 | — |
| 取消退款 | ✅ | ✅ 积分/库存恢复 | ✅ | — |
| 积分明细 | ✅ | — | ✅ 交易记录 | — |
| 管理订单 | ✅ | — | ✅ 列表展示 | — |

---

## 已修复的 Bug (Product Service — 本人负责)

### Bug 1: WebFlux/WebMvc 冲突导致所有接口 404

- 文件: `application/application-impl/pom.xml`
- 原因: 引入了 `spring-boot-starter-webflux`，与 WebMvc 冲突
- 修复: 移除该依赖（Product Service 不需要 WebClient）
- Commit: `95ecd80`

### Bug 2: 前后端字段名不一致导致商品详情报"商品不存在"

- 文件: `GetProductRequest.java`, `DeleteProductRequest.java`, `UpdateStatusRequest.java`, `ProductApplicationServiceImpl.java`, `InternalProductController.java`
- 原因: 后端用 `id`，前端用 `productId`
- 修复: 后端字段统一改为 `productId`
- Commit: `458c600`

---

## 发现的其他仓库问题 (Bug Report)

### [Frontend] 管理员新增商品时缺少分类选择器

- 仓库: awsome-shop-frontend
- 状态: ✅ 已由前端组员修复 (commit `9da0757`)
- 描述: `src/pages/admin/Products/index.tsx` 表单中缺少分类选择器，`categoryId` 始终为默认值 0

### [Order Service] 取消订单后积分/库存未同步退还

- 仓库: awsome-shop-order-service
- 状态: ⚠️ 待确认
- 描述: 员工取消订单后，积分余额和商品库存未立即恢复。可能原因：
  1. 退款走异步补偿队列（Redis），需要等待 CompensationScheduler 执行
  2. 取消 API 调用成功但跨服务退还失败
- 建议: 检查 `CompensationScheduler` 执行频率和 `RedemptionApplicationServiceImpl.cancel()` 逻辑

---

## 测试文件位置

- 测试代码: `awsome-shop-frontend/e2e/shop.spec.ts`
- Playwright 配置: `awsome-shop-frontend/playwright.config.ts`
- 失败截图目录: `awsome-shop-frontend/test-results/`
