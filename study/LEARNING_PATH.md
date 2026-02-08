# eShop 后端深度学习路径 (Backend Deep Dive)

这份学习计划专为希望**深入掌握** .NET 云原生后端开发的初学者设计。我们完全跳过前端部分，专注于架构、数据、通信和微服务设计模式。

目标：从“会写代码”进阶到“理解架构”。

---

## 📅 第一阶段：环境与全局架构 (The Big Picture)

在深入代码之前，你需要理解“它是怎么跑起来的”。eShop 使用了最新的 **.NET Aspire** 进行编排，这与传统的 Docker Compose 有所不同。

### 1. 运行与调试基础
- **目标**：能自如地启动、停止、重置环境，并看懂日志。
- **任务**：
    - [ ] 阅读 `src/eShop.AppHost/Program.cs`。这是整个分布式系统的“入口”。理解它是如何定义 Postgres, Redis, RabbitMQ 容器的。
    - [ ] 启动项目，访问 **Aspire Dashboard**。熟悉“Projects”（服务）、“Containers”（中间件）、“Traces”（调用链）三个面板。
    - [ ] 尝试修改 `AppHost` 配置，例如改变 Redis 的端口，观察发生了什么。

### 2. Service Defaults (服务默认配置)
- **目标**：理解微服务如何做到“开箱即用”的健康检查与服务发现。
- **核心代码**：`src/eShop.ServiceDefaults` 项目。
- **任务**：
    - [ ] 研究 `Extensions.cs` 中的 `AddServiceDefaults` 方法。
    - [ ] 理解 OpenTelemetry 是如何被配置的（日志、以及分布式追踪）。
    - [ ] 弄懂 `AddDefaultHealthChecks` 是如何工作的。

---

## 🔨 第二阶段：微服务基础与数据驱动 (Catalog.API)

`Catalog.API` 是最标准的微服务，负责商品管理。它结构相对简单，但包含了现代 AI 功能。

### 1. EF Core 与关系型数据库
- **目标**：掌握 Code-First 开发模式。
- **核心代码**：`src/Catalog.API/Infrastructure/*`.
- **任务**：
    - [ ] 分析 `CatalogContext`。理解 `DbContext` 如何映射实体。
    - [ ] 学习 **EntityConfiguration**。为什么不使用 Attribute 而使用 Fluent API 配置数据库字段？
    - [ ] 运行数据库迁移命令，理解 `dotnet ef migrations` 的作用。

### 2. REST API 设计
- **目标**：编写规范的 HTTP 接口。
- **核心代码**：`src/Catalog.API/Apis/CatalogApi.cs`.
- **任务**：
    - [ ] 学习 Minimal API 的写法（`app.MapGroup`）。
    - [ ] 理解分页逻辑（Pagination）是如何实现的。

### 3. AI 与向量数据库 (进阶)
- **目标**：这是 eShop 的亮点。理解如何做“语义搜索”。
- **核心代码**：`Catalog.AI` 相关代码。
- **任务**：
    - [ ] 找出 `Pgvector` 是如何被使用的。
    - [ ] 理解商品图片/描述是如何被转化为 Vector（向量）并存入 PostgreSQL 的。

---

## 🛒 第三阶段：中间件与高性能存储 (Basket.API)

购物车服务（Basket）引入了 **Redis** 和 **gRPC**，与 Catalog 的 SQL 模式完全不同。

### 1. 分布式缓存 (Redis)
- **目标**：理解 Key-Value 存储与持久化差异。
- **核心代码**：`src/Basket.API/Storage/RedisBasketRepository.cs`.
- **任务**：
    - [ ] 比较 Redis 存储与 PostgreSQL 存储代码的区别。
    - [ ] 思考：如果 Redis 挂了，购物车数据还在吗？eShop 是怎么处理的？

### 2. gRPC 服务间通信
- **目标**：理解高性能内部通信。
- **核心代码**：`src/Basket.API/Grpc/BasketService.cs`.
- **任务**：
    - [ ] 查看 `.proto` 文件定义。
    - [ ] 比较 gRPC 方法与 REST API 方法的异同。
    - [ ] 为什么 Basket 和 Ordering 之间要用 gRPC？

---

## 🏗 第四阶段：领域驱动设计与复杂业务 (Ordering.API)

这是最难也是最精华的部分。`Ordering` 服务展示了如何处理复杂的业务逻辑。

### 1. DDD 分层架构
- **目标**：理解为什么要把一个项目拆成 API, Domain, Infrastructure 三个库。
- **任务**：
    - [ ] **Domain 层**：查看 `Ordering.Domain/AggregatesModel`。理解什么是我们可以“聚合根”（Aggregate Root）。为什么 `Order` 是聚合根而 `OrderItem` 不是？
    - [ ] **Infrastructure 层**：查看 Repository 的实现。它仅负责存取，不负责业务逻辑。

### 2. CQRS (命令查询职责分离)
- **目标**：将“写操作”（Command）与“读操作”（Query）分开。
- **核心代码**：`src/Ordering.API/Application/Commands` vs `Queries`.
- **任务**：
    - [ ] 追踪通过 MediatR 发送的一个 Command（例如 `CreateOrderCommand`）。
    - [ ] 比较 Query 逻辑（直接写 SQL/Dapper）与 Command 逻辑（走 EF Core Repository）的区别。

### 3. 最终一致性与集成事件
- **目标**：确保分布式事务的一致性。
- **核心代码**：`IntegrationEventLogEF` 和 `RabbitMQ`。
- **任务**：
    - [ ] **发件箱模式 (Outbox Pattern)**：学习它是如何保证“订单创建成功”和“发送消息给 RabbitMQ”这两件事要么都成功，要么都失败的。
    - [ ] 追踪 `OrderStartedIntegrationEvent`。它从哪里发出？谁消费了它？（提示：Basket 会消费它来清空购物车）。

---

## 🛡 第五阶段：安全与企业级特性 (Identity & Security)

最后，我们需要保护我们的 API。

### 1. 身份认证与授权 (Duende IdentityServer)
- **目标**：理解 OAuth2 和 OIDC 流程。
- **核心代码**：`src/Identity.API`.
- **任务**：
    - [ ] 理解 Token（令牌）是如何生成的。
    - [ ] 在 `Ordering.API` 中找到 `[Authorize]` 属性，理解它是如何验证 Token 的。

### 2. 韧性与容错 (Resilience)
- **目标**：构建打不死的服务。
- **逻辑**：如果数据库闪断了一下，服务会崩溃吗？
- **任务**：
    - [ ] 寻找 **Polly** 策略配置（重试、熔断）。
    - [ ] 在 `HttpRetries` 相关配置中查看它是如何处理网络抖动的。
