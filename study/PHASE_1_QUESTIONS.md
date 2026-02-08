# 第一阶段自测题：环境与架构 (Phase 1: Environment & Architecture)

这些问题旨在检验你对 eShop 全局架构和 .NET Aspire 编排的理解。请先尝试回答，再点击展开查看答案。

---

## 🏗️ 1. AppHost 编排 (Orchestration)

### Q1: 各种“数据库”容器是如何被定义的？
在 `AppHost` 中，并不需要写 Docker Compose 文件来定义 Redis 或 Postgres。请找到定义 Redis 和 RabbitMQ 容器的代码。

👉 **[点击这里查看代码](file:///Users/xianzhezhang/Projects/eShop/src/eShop.AppHost/Program.cs#L7-L13)**

<details>
<summary>👀 <b>点击查看答案</b></summary>

使用了 `.NET Aspire` 的 `IDistributedApplicationBuilder` API：
- `builder.AddRedis("redis")` 定义了 Redis 容器。
- `builder.AddRabbitMQ("eventbus")` 定义了 RabbitMQ 容器。
- `builder.AddPostgres("postgres")` 定义了 Postgres 数据库容器。

`.WithLifetime(ContainerLifetime.Persistent)` 保证了重启应用时数据不会丢失（容器数据卷持久化）。
</details>

---

### Q2: 微服务之间是如何“认识”的？
`Catalog.API` 需要连接数据库，也需要连接 RabbitMQ。它是如何获取到这些连接信息的？请找到相关代码。

👉 **[点击这里查看代码](file:///Users/xianzhezhang/Projects/eShop/src/eShop.AppHost/Program.cs#L35-L37)**

<details>
<summary>👀 <b>点击查看答案</b></summary>

通过 `.WithReference(...)` 方法。
```csharp
var catalogApi = builder.AddProject<Projects.Catalog_API>("catalog-api")
    .WithReference(rabbitMq)
    .WithReference(catalogDb);
```
这会自动将 RabbitMQ 和 Database 的连接字符串注入到 `Catalog.API` 的环境变量中。Aspire 会自动处理端口映射和连接字符串生成。
</details>

---

### Q3: 身份认证服务 (Identity API) 的地址是如何传递给购物篮服务 (Basket API) 的？
Basket API 需要知道 Identity API 的地址来进行令牌验证。这部分配置在哪里？

👉 **[点击这里查看代码](file:///Users/xianzhezhang/Projects/eShop/src/eShop.AppHost/Program.cs#L29-L32)**

<details>
<summary>👀 <b>点击查看答案</b></summary>

通过显式设置环境变量 `Identity__Url`：
```csharp
.WithEnvironment("Identity__Url", identityEndpoint);
```
这里 `identityEndpoint` 是通过 `identityApi.GetEndpoint(launchProfileName)` 动态获取的。注意双下划线 `__` 在 .NET 配置中代表层级（即 `Identity:Url`）。
</details>

---

## 🛠️ 2. Service Defaults (服务默认配置)

### Q4: 什么是 Service Defaults？它包含了哪些默认功能？
所有微服务都调用了 `builder.AddServiceDefaults()`。这个方法到底干了什么？

👉 **[点击这里查看代码](file:///Users/xianzhezhang/Projects/eShop/src/eShop.ServiceDefaults/Extensions.cs#L16-L32)**

<details>
<summary>👀 <b>点击查看答案</b></summary>

`AddServiceDefaults` 封装了所有微服务共用的基础能力，主要包括：
1. **OpenTelemetry** (`ConfigureOpenTelemetry`): 配置日志、指标（Metrics）和分布式追踪（Tracing）。
2. **Health Checks** (`AddDefaultHealthChecks`): 添加健康检查端点。
3. **Service Discovery** (`AddServiceDiscovery`): 允许使用服务名（如 `http://catalog-api`）而不是 IP 地址来调用服务。
4. **Resilience** (`AddStandardResilienceHandler`): 为 HTTP 客户端默认开启重试、超时等韧性机制。
</details>

---

### Q5: 健康检查 (Health Checks) 端点是在哪里映射的？
如果我想知道一个服务是否存活，应该访问哪个 URL？

👉 **[点击这里查看代码](file:///Users/xianzhezhang/Projects/eShop/src/eShop.ServiceDefaults/Extensions.cs#L108-L128)**

<details>
<summary>👀 <b>点击查看答案</b></summary>

在 `MapDefaultEndpoints` 方法中映射了：
- `/health`: 所有检查都通过才返回 200 OK（用于就绪探针 Readiness Probe）。
- `/alive`: 只要应用进程活着就返回 200 OK（用于存活探针 Liveness Probe）。

注意：出于安全考虑，这些端点在非开发环境（Production）默认可能只有部分开启或需要鉴权（虽然示例代码中有一段注释说明了这一点，第 115 行限制了 `IsDevelopment` 才完全暴露简单端点）。
</details>

---

## 🎯 挑战任务 (Challenge)

尝试修改 `src/eShop.AppHost/Program.cs`，将 Redis 的资源名称从 `"redis"` 改为 `"my-redis"`。

**思考**：
1. 这里改了之后，`Basket.API` 里还能连上 Redis 吗？
2. 如果不能，Basket API 的代码（`src/Basket.API/Program.cs`）里可能需要对应修改什么？（提示：连接字符串的名字通常和服务名/资源名有关）。
