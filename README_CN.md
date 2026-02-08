# eShop 项目说明文档

本项目是一个基于 **.NET Aspire** 的现代电子商务参考应用，展示了云原生、微服务架构下的最佳实践。

## 🚀 核心技术栈

| 技术 | 作用 |
| :--- | :--- |
| **.NET 10** | 核心开发框架，提供高性能的服务端与客户端运行环境。 |
| **.NET Aspire** | 云原生应用编排工具，用于管理微服务的启动、依赖、部署及可观测性。 |
| **Microservices** | 采用微服务架构，实现模块化开发、独立部署与水平扩展。 |
| **EF Core + PostgreSQL** | 关系型数据库访问，用于持久化存储商品、订单、用户信息等。 |
| **Redis** | 提供分布式缓存，主要用于购物车（Basket）数据的快速读写。 |
| **RabbitMQ** | 消息中间件，作为事件总线（Event Bus）实现微服务间的异步解耦通信。 |
| **Duende IdentityServer** | 身份认证与授权中心，支持 OIDC/OAuth 2.0 协议。 |
| **Blazor (Web)** | 用于构建交互式 Web 前端界面。 |
| **.NET MAUI (Client)** | 跨平台移动和桌面端应用开发框架。 |
| **OpenTelemetry** | 集成化的日志、指标和追踪，通过 Aspire Dashboard 实时监控系统运行状态。 |
| **gRPC & REST** | 服务间高效的同步通信方式。 |

---

## 📂 目录结构及功能说明

### `src/` (核心源代码)

#### 后端微服务
- **`Basket.API`**: 购物车微服务。负责管理用户的临时购物车数据，具有高性能的缓存存储特点。
- **`Catalog.API`**: 商品目录微服务。负责商品信息的 CRUD、品牌管理及分类，是商城的“货架”。
- **`Ordering.API`**: 订单微服务。处理下单逻辑、支付回调处理及订单状态流转。
- **`Identity.API`**: 身份授权微服务。基于 ASP.NET Core Identity，处理用户登录、注册及令牌发放。
- **`Webhooks.API`**: 负责向订阅了特定事件（如订单完成）的外部系统发送通知。

#### 领域与基础设施
- **`Ordering.Domain`**: 订单系统的核心领域逻辑。采用 DDD（领域驱动设计），包含实体、聚合根及业务领域规则。
- **`Ordering.Infrastructure`**: 订单系统的基础设施实现。包含数据库映射、持久化逻辑及外部系统适配。
- **`IntegrationEventLogEF`**: 集成事件日志。用于实现“发件箱模式”（Outbox Pattern），确保消息发送与数据库操作的最终一致性。

#### 通信与调度
- **`EventBus` / `EventBusRabbitMQ`**: 事件总线抽象及 RabbitMQ 实现。支撑服务间的异步消息传递。
- **`PaymentProcessor`**: 支付处理后台应用。监听支付事件并模拟真实的支付流程。
- **`OrderProcessor`**: 订单处理后台任务，负责背景业务逻辑的处理。

#### 前端与客户端
- **`WebApp`**: 基于 Blazor 的标准 Web 前端，直接面向浏览器用户。
- **`ClientApp`**: 基于 .NET MAUI 的原生跨平台客户端（Android, iOS, Windows, macOS）。
- **`WebhookClient`**: 一个简单的 Webhook 接收客户端示例，展示如何消费商城发出的通知。

#### 编排与基础设施
- **`eShop.AppHost`**: **核心编排项目**。用于定义 PostgreSQL、Redis、RabbitMQ 以及各 API 服务的依赖关系，是本地启动的入口。
- **`eShop.ServiceDefaults`**: 共享的服务默认项。封装了分布式系统常用的韧性（Resilience）、健康检查、服务发现等通用配置。

### 辅助目录
- **`tests/`**: 包含各单元测试（Unit Test）和集成测试（Integration Test）项目。
- **`e2e/`**: 端到端测试目录，使用 **Playwright** 进行全链路流程验证。
- **`.aspire/`**: 存放 .NET Aspire 相关的元数据和本地运行配置。
- **`img/`**: 存放项目所需的静态图片资源。

---

## 🛠 如何开始

1. 确保安装了最新版本的 **.NET SDK**。
2. 安装 **.NET Aspire 工作负载** (`dotnet workload install aspire`)。
3. 确保本地安装并运行了 **Docker Desktop**（用于启动数据库和中间件容器）。
4. 在根目录运行命令或在 IDE 中启动 `eShop.AppHost`：
   ```bash
   dotnet run --project src/eShop.AppHost
   ```
5. 启动后，通过控制台输出的链接访问 **Aspire Dashboard** 查看所有服务的状态。
