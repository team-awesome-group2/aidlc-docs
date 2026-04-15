# Requirements Verification Questions

基于逆向工程分析和您的开发意图（完善现有功能 + Phase 1 核心闭环 + Phase 2 管理能力），请回答以下问题以明确需求范围。

请在每个问题的 `[Answer]:` 后填写您选择的字母。

---

## Question 1

Auth Service 的用户注册方式？

A) 仅管理员后台创建用户（员工不能自行注册）
B) 员工可以自行注册，管理员也可以后台创建
C) 对接企业 SSO/LDAP，不需要独立注册功能
D) Other (please describe after [Answer]: tag below)

[Answer]: B

## Question 2

积分发放的触发方式？

A) 仅管理员手动发放（单个/批量）
B) 管理员手动发放 + 系统自动发放（如每月固定额度）
C) 管理员手动发放 + 基于规则自动发放（如完成任务/绩效奖励）
D) Other (please describe after [Answer]: tag below)
[Answer]: B

## Question 3

订单兑换流程中，跨服务的数据一致性如何处理？（兑换需要同时：扣积分 + 扣库存 + 创建订单）

A) 简单方案：同步 HTTP 调用，失败时手动补偿（适合 MVP 阶段）
B) 本地事务 + 异步消息最终一致性（利用已有的 SQS 模块）
C) Saga 模式：编排式分布式事务（更可靠但更复杂）
D) Other (please describe after [Answer]: tag below)

[Answer]: 本地事务+ 基于Redis实现队列或者其他本地队列方案

## Question 4

商品分类管理的复杂度？

A) 一级分类即可（如：数码电子、生活家居、美食餐饮...）
B) 支持多级分类（如：数码电子 > 手机 > 智能手机）
C) Other (please describe after [Answer]: tag below)

## Question 5

管理员仪表盘需要哪些统计数据？

A) 基础统计：商品总数、用户总数、本月兑换量、积分流通量
B) 基础统计 + 趋势图表（近7天/30天兑换趋势、积分消耗趋势）
C) 基础统计 + 趋势图表 + 排行榜（热门商品 Top10、活跃用户 Top10）
D) Other (please describe after [Answer]: tag below)

## Question 6

前端对接后端 API 时，是否需要同时处理 Token 刷新机制？

A) 是，Token 过期前自动刷新（无感刷新）
B) 否，Token 过期后跳转登录页重新登录即可（当前 2 小时过期）
C) Other (please describe after [Answer]: tag below)

## Question 7

现有 API 全部使用 POST 方法，是否在本次开发中改为 RESTful 风格（GET/POST/PUT/DELETE）？

A) 是，新增和修改的 API 都改为 RESTful 风格
B) 否，保持现有 POST-only 风格，保持一致性
C) 仅新增的 API 使用 RESTful，已有的保持不变
D) Other (please describe after [Answer]: tag below)

## Question 8

订单取消后的积分退还和库存恢复策略？

A) 允许取消，立即退还积分和恢复库存
B) 允许取消，但有时间限制（如下单后 24 小时内可取消）
C) 不允许取消，兑换后不可撤销
D) Other (please describe after [Answer]: tag below)

## Question 9

本次开发是否需要编写单元测试和集成测试？

A) 是，核心业务逻辑需要单元测试 + 关键 API 需要集成测试
B) 仅核心业务逻辑的单元测试
C) 暂不需要测试，后续补充
D) Other (please describe after [Answer]: tag below)

## Question 10: Security Extensions

Should security extension rules be enforced for this project?

A) Yes — enforce all SECURITY rules as blocking constraints (recommended for production-grade applications)
B) No — skip all SECURITY rules (suitable for PoCs, prototypes, and experimental projects)
C) Other (please describe after [Answer]: tag below)

## Question 11: Property-Based Testing Extension

Should property-based testing (PBT) rules be enforced for this project?

A) Yes — enforce all PBT rules as blocking constraints (recommended for projects with business logic, data transformations, serialization, or stateful components)
B) Partial — enforce PBT rules only for pure functions and serialization round-trips (suitable for projects with limited algorithmic complexity)
C) No — skip all PBT rules (suitable for simple CRUD applications, UI-only projects, or thin integration layers with no significant business logic)
D) Other (please describe after [Answer]: tag below)
