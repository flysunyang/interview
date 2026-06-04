# Spring Cloud 面试题

## 1、什么是 Spring Cloud？

**满分答案：**
Spring Cloud 是一套基于 Spring Boot 的**微服务解决方案**，它为开发者提供了在分布式系统中快速构建**常见模式**的工具（如服务注册与发现、配置管理、断路器、负载均衡、API 网关等）。

**核心特点：**
- **基于 Spring Boot**：自动配置，开箱即用
- **分布式/微服务生态**：一站式微服务治理方案
- **组件丰富**：涵盖服务治理、配置管理、熔断、路由等各个方面
- **生态整合**：可以与 Netflix OSS、Alibaba、Consul 等第三方组件无缝集成

**Spring Cloud 全家桶主要组件：**

| 组件           | 功能             | 常用实现                                        |
| -------------- | ---------------- | ----------------------------------------------- |
| 服务注册与发现 | 管理服务实例地址 | Eureka / Nacos / Consul / Zookeeper             |
| 服务调用       | 微服务间远程调用 | OpenFeign / RestTemplate / Dubbo                |
| 负载均衡       | 请求分发         | Ribbon / Spring Cloud LoadBalancer              |
| 熔断降级       | 防止级联故障     | Hystrix / Resilience4j / Sentinel               |
| API 网关       | 统一入口、路由   | Spring Cloud Gateway / Zuul                     |
| 配置中心       | 统一配置管理     | Spring Cloud Config / Nacos / Apollo            |
| 链路追踪       | 调用链跟踪       | Sleuth + Zipkin / Skywalking                    |
| 消息总线       | 配置刷新、通信   | Spring Cloud Bus                                |
| 分布式事务     | 跨服务事务       | Seata                                           |
| 远程调用       | 服务间通信       | OpenFeign / Dubbo / gRPC                        |

---

## 2、Spring Cloud 和 Spring Boot 的关系？

**满分答案：**

| 对比项     | Spring Boot                              | Spring Cloud                                    |
| ---------- | ---------------------------------------- | ----------------------------------------------- |
| 定位       | 快速构建**单体应用**的脚手架             | 构建**分布式/微服务系统**的一站式解决方案       |
| 核心功能   | 自动配置、起步依赖、内嵌容器、Actuator等 | 服务注册/发现、配置中心、熔断、网关、负载均衡等 |
| 依赖关系   | 独立使用                                 | **基于 Spring Boot**，不能脱离 Spring Boot      |
| 版本对应   | 独立版本号（如 2.7.x、3.x）              | 有对应的 Release Train 版本（Hoxton、2021.x 等） |

> **一句话总结**：Spring Boot 是盖楼的砖块和水泥，Spring Cloud 是盖楼的规划和框架。没有 Spring Boot 就没有 Spring Cloud。

---

## 3、说说你对微服务的理解？

**满分答案：**

微服务是一种**架构风格**，将单一应用拆分为一组**小型、独立的服务**，每个服务围绕特定业务能力构建，可以独立开发、部署、扩展。

**微服务的优势：**
- **独立部署**：修改一个服务不影响其他服务
- **技术异构**：不同服务可以选择不同的技术栈和数据库
- **可扩展性强**：针对性地对热点服务进行水平扩展
- **团队自治**：服务边界对应组织边界，小团队独立负责
- **容错性**：一个服务宕机不会导致整个系统崩溃

**微服务的挑战：**
- 分布式复杂性（网络延迟、分布式事务、数据一致性）
- 运维成本高（需要容器化、CI/CD、监控、链路追踪等基础设施）
- 服务间通信（RPC vs HTTP，序列化开销）
- 接口管理与版本控制
- 分布式数据管理（CAP 理论的取舍）

**微服务 vs 单体架构：**

| 对比项     | 单体架构                   | 微服务架构                     |
| ---------- | -------------------------- | ------------------------------ |
| 部署       | 整体打包部署               | 每个服务独立部署               |
| 扩展       | 整体扩展                 | 按服务粒度扩展                 |
| 技术栈     | 统一                 | 可异构                 |
| 数据库     | 通常单库             | 每个服务独享数据库（数据去中心化） |
| 故障隔离   | 一处崩溃全局受影响           | 故障隔离，不会级联             |
| 团队规模   | 适合小团队                 | 适合多团队协同                 |

---

## 4、CAP 理论是什么？Spring Cloud 组件如何选择？

**满分答案：**

CAP 理论是分布式系统的基石：一致性（Consistency）、可用性（Availability）、分区容错性（Partition Tolerance），三者最多只能同时满足两个。

**三个维度的含义：**
- **C（一致性）**：所有节点在同一时间看到相同的数据
- **A（可用性）**：每个请求都能收到成功或失败的响应（系统始终可用）
- **P（分区容错性）**：系统在网络分区故障时仍能正常运作

> **关键理解**：分布式系统中 P（分区容错性）是必须保证的，因此实际是在 CP（强一致性）和 AP（高可用性）之间做选择。

**Spring Cloud 组件的 CAP 选择：**

| 组件       | CAP 类型 | 说明                                           |
| ---------- | -------- | ---------------------------------------------- |
| Eureka     | AP       | 优先保证可用性，节点平等，无主从，适合高可用场景 |
| Zookeeper  | CP       | 保证强一致性，Leader 选举期间不可用             |
| Consul     | CP       | 强一致性，基于 Raft 协议                       |
| Nacos      | AP/CP    | 可切换，默认 AP                                |

---

## 5、什么是服务注册中心？Eureka 和 Nacos 的区别？

**满分答案：**

服务注册中心是微服务架构中的"**通讯录**"，每个服务启动时将自身地址注册到注册中心，消费者从注册中心获取服务地址列表，实现服务发现。

**核心功能：**
- **服务注册**：服务提供者启动时向注册中心注册
- **服务发现**：服务消费者从注册中心获取可用服务列表
- **健康检查**：定期检测服务实例是否存活
- **服务下线**：服务停止时自动从注册中心移除

**Eureka vs Nacos：**

| 对比项       | Eureka                            | Nacos                                    |
| ------------ | --------------------------------- | ---------------------------------------- |
| CAP 模型     | AP（高可用）                      | AP + CP（可切换）                        |
| 健康检查     | 客户端心跳（续约机制）            | TCP/HTTP/MySQL 多种方式                  |
| 配置中心     | 不支持，需配合 Spring Cloud Config | **自带配置中心**                         |
| 自我保护     | 有自我保护机制                    | 无自我保护（但有健康检查）               |
| 一致性协议   | 无（Peer to Peer 复制）           | Distro（AP）/ Raft（CP）                 |
| 负载均衡     | 自带 Ribbon 集成                  | 自带权重/负载均衡                        |
| 功能丰富度   | 仅服务注册/发现                   | 服务发现 + 配置管理 + DNS                |
| 维护状态     | **已停更（2.x）**                 | **活跃维护（推荐）**                     |

> **当前趋势**：Eureka 已停止维护，新项目推荐使用 Nacos 或 Consul。

---

## 6、Eureka 的自我保护机制是什么？

**满分答案：**

Eureka 自我保护机制是一种**防止误剔除**的容错设计。当 Eureka Server 短时间内丢失大量客户端心跳时，会进入**自我保护模式**，不再剔除任何服务实例。

**触发条件：**
- 实际每分钟续约数量 < 期望每分钟续约数量的 **85%**（默认阈值）

**为什么需要自我保护：**
- 网络波动导致 Eureka Server 暂时收不到心跳，但服务实例本身是健康的
- 如果此时剔除了"假死"的服务，会导致服务不可用

**自我保护 vs 剔除：**

| 场景           | 行为                         |
| -------------- | ---------------------------- |
| 网络波动       | 进入自我保护，**保留**所有实例 |
| 服务确实宕机   | 正常剔除过期实例（15 分钟 + 90 秒） |

> **CAP 的体现**：自我保护机制是 Eureka 选择 AP（高可用）的体现——宁可保留故障节点也不误删健康节点。

---

## 7、Ribbon 和 Spring Cloud LoadBalancer 的区别？

**满分答案：**

两者都是**客户端负载均衡器**，工作在服务消费者端。

**核心概念：**
- **客户端负载均衡**：服务消费者在本地维护服务地址列表，自行选择目标实例发起调用
- **服务端负载均衡**：由独立的负载均衡器（如 Nginx）统一分发请求

**Ribbon vs Spring Cloud LoadBalancer：**

| 对比项           | Ribbon                      | Spring Cloud LoadBalancer     |
| ---------------- | --------------------------- | ----------------------------- |
| 维护状态         | **已停更（进入维护模式）**  | **官方推荐，活跃维护**        |
| 负载均衡策略     | 轮询、随机、加权、最小并发等 | 轮询、随机（可自定义）        |
| 与 RestTemplate 集成 | 原生支持                  | 原生支持（需引入 spring-cloud-loadbalancer） |
| Nacos 权重支持   | 需额外配置                  | 原生支持 Nacos 权重           |
| Spring Cloud 版本 | Hoxton 及之前              | 2020.0 及之后                 |

**常见负载均衡策略（Ribbon）：**
- `RoundRobinRule`：轮询（默认）
- `RandomRule`：随机
- `WeightedResponseTimeRule`：根据响应时间分配权重
- `BestAvailableRule`：选择并发最小的服务
- `RetryRule`：失败重试
- `AvailabilityFilteringRule`：过滤掉故障实例
- `ZoneAvoidanceRule`：区域感知

> **当前实践**：Spring Cloud 2020.0 开始官方推荐使用 Spring Cloud LoadBalancer 替代 Ribbon。

---

## 8、什么是服务雪崩？如何解决？

**满分答案：**

**服务雪崩**：在微服务调用链中，一个服务故障引发**级联故障**，导致整个系统不可用。

```
A 调用 B → B 调用 C → C 响应慢/不可用
→ B 线程堆积 → B 资源耗尽 → B 不可用
→ A 调用 B 超时 → A 线程堆积 → 整个系统崩溃
```

**解决方案（熔断、降级、限流）：**

| 方案       | 说明                                       | 类比               |
| ---------- | ------------------------------------------ | ------------------ |
| **熔断**   | 达到故障阈值后直接拒绝请求，快速失败       | 保险丝熔断         |
| **降级**   | 服务不可用时返回备选方案（兜底数据）       | 备选方案           |
| **限流**   | 控制单位时间内的请求量，超过则排队或拒绝   | 排队限流           |
| **隔离**   | 线程池/信号量隔离，防止一个服务拖垮整个系统 | 船舱隔板隔离       |
| **超时**   | 设置调用超时时间，避免无限等待             | 请求超时           |

**代码示例（Sentinel 熔断降级）：**
```java
@SentinelResource(value = "getUser", fallback = "fallbackMethod")
public User getUser(Long id) {
    return userService.getUser(id); // 可能超时或异常
}

// 降级方法：返回兜底数据
public User fallbackMethod(Long id, Throwable e) {
    return new User(id, "默认用户");
}
```

---

## 9、Hystrix、Resilience4j 和 Sentinel 的区别？

**满分答案：**

三者都是**熔断降级框架**。

| 对比项       | Hystrix                     | Resilience4j              | Sentinel                               |
| ------------ | --------------------------- | ------------------------- | -------------------------------------- |
| 维护状态     | **已停更**                  | **活跃维护**              | **活跃维护（阿里开源）**               |
| 实现方式     | 基于线程池/信号量隔离       | 基于装饰器模式、函数式    | 基于统计滑动窗口                       |
| 熔断策略     | 异常比例                    | 异常比例 + 慢调用比例     | 异常比例 + 慢调用比例 + 异常数         |
| 限流         | 不支持（需配合其他组件）    | 支持（RateLimiter）       | **支持（QPS/线程数/热点/系统规则）**   |
| 控制台       | Hystrix Dashboard           | 无自带（可集成 Prometheus）| **Sentinel Dashboard（可视化配置）**   |
| 规则持久化   | 不支持                      | 不支持                    | **支持（推送到 Nacos/Apollo）**        |
| Spring Cloud集成 | Netflix Hystrix          | Spring Cloud Circuit Breaker | Spring Cloud Alibaba Sentinel        |

> **当前推荐**：国内项目推荐 Sentinel（功能最全面，可视化控制台），国外项目推荐 Resilience4j。

---

## 10、Spring Cloud Gateway 和 Zuul 的区别？

**满分答案：**

API 网关是微服务的**统一入口**，负责路由转发、鉴权、限流、日志、跨域等。

| 对比项       | Zuul 1.x               | Zuul 2.x                     | Spring Cloud Gateway             |
| ------------ | ---------------------- | ---------------------------- | -------------------------------- |
| 底层实现     | **Servlet（阻塞式）**  | Netty（非阻塞）              | **Netty + WebFlux（响应式）**   |
| 线程模型     | 同步阻塞，每请求一线程 | 异步非阻塞                   | 异步非阻塞                       |
| 性能         | 较低                   | 高                           | 高                               |
| 长连接支持   | 不支持 WebSocket       | 支持                         | 支持 WebSocket                   |
| Spring 版本  | Spring Boot 1.x/2.x    | Spring Boot 2.x              | Spring Boot 2.x/3.x              |
| 维护状态     | **已停更**             | Netflix 内部停止开源维护     | **Spring 官方维护，推荐**        |

**Spring Cloud Gateway 三大核心概念：**

| 概念     | 说明                                     |
| -------- | ---------------------------------------- |
| Route    | 路由：ID + 目标 URI + Predicates + Filters |
| Predicate| 断言：匹配请求的条件（Path、Header、Method 等）|
| Filter   | 过滤器：对请求/响应进行修改、鉴权、限流等     |

**Gateway 配置示例：**
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://user-service
          predicates:
            - Path=/api/user/**
          filters:
            - StripPrefix=1
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
```

---

## 11、OpenFeign 是什么？它的工作原理？

**满分答案：**

OpenFeign 是一个**声明式的 HTTP 客户端**，让远程调用像调用本地方法一样简单。它集成了 Ribbon/LoadBalancer 实现负载均衡。

**核心特性：**
- **声明式调用**：只需定义接口 + 注解，不用手写 HTTP 请求代码
- **负载均衡**：内部集成 Ribbon/LoadBalancer
- **熔断降级**：集成 Hystrix/Sentinel 作为 fallback
- **请求拦截**：支持 RequestInterceptor 统一添加请求头（如 Token）
- **编解码**：集成 Spring MVC 注解（@RequestParam、@RequestBody）
- **压缩**：支持 GZIP 请求/响应压缩

**工作流程：**
1. 通过 `@FeignClient` 注解声明接口
2. Spring 容器启动时扫描 @FeignClient → 生成 JDK 动态代理
3. 调用接口方法时 → 代理对象根据方法上的注解构建 HTTP 请求
4. 从服务注册中心获取服务地址列表
5. 通过负载均衡选择目标实例
6. 发送 HTTP 请求 → 解析响应 → 返回结果

**代码示例：**
```java
@FeignClient(name = "user-service", fallback = UserServiceFallback.class)
public interface UserServiceClient {
    @GetMapping("/user/{id}")
    User getUser(@PathVariable("id") Long id);
    
    @PostMapping("/user")
    Result addUser(@RequestBody User user);
}
```

---

## 12、Spring Cloud Config 是如何实现配置刷新的？

**满分答案：**

Spring Cloud Config 提供**统一的配置管理**，支持将配置存储在 Git / SVN / 本地文件系统中。

**配置刷新方式：**

| 方式                      | 说明                                           | 是否需重启 |
| ------------------------- | ---------------------------------------------- | ---------- |
| 手动调用 `/actuator/refresh` | 单个节点调用 POST /actuator/refresh            | 否         |
| Spring Cloud Bus          | 通过消息总线广播刷新事件，所有节点自动刷新     | 否         |
| Nacos/Apollo              | 自动监听配置变更并推送                         | 否         |

**Spring Cloud Bus 刷新流程：**
```
1. 修改 Git 配置
2. POST /actuator/busrefresh (任意一个节点)
3. 该节点通过消息总线（RabbitMQ/Kafka）广播 RefreshRemoteApplicationEvent
4. 所有订阅节点收到事件 → 重新加载配置（@RefreshScope 的 Bean 重新初始化）
```

**@RefreshScope 原理：**
- 标注了 `@RefreshScope` 的 Bean 会被放入一个**懒加载代理**中
- 刷新时销毁原 Bean 的缓存 → 下次访问时重新初始化 → 获取最新配置

> **当前实践**：Nacos/Apollo 比 Spring Cloud Config + Bus 更轻量和易用，新项目推荐使用。

---

## 13、Nacos 是如何实现服务注册和配置管理一体化的？

**满分答案：**

Nacos（Dynamic Naming and Configuration Service）是阿里巴巴开源的**动态服务发现、配置管理和服务管理平台**。

**服务注册原理：**
1. 服务启动 → 向 Nacos Server 发送注册请求（POST /nacos/v1/ns/instance）
2. Nacos Server 将实例信息存储到**内存注册表** + 持久化到数据库
3. 服务消费者启动 → 向 Nacos Server 定时拉取（Pull）服务列表
4. Nacos Server 主动推送（Push）变更通知 → 消费者更新本地缓存
5. 服务提供者定时发送心跳（默认 5s）→ Nacos 检测健康状态

**配置管理原理：**
1. 应用启动 → 从 Nacos Server 拉取配置
2. Nacos 客户端开启长轮询（Long Polling）监听配置变更
3. 配置变更 → Nacos Server 推送变更通知
4. 客户端收到通知 → 更新本地配置 → 刷新标记了 @RefreshScope 的 Bean

**Nacos vs Spring Cloud Config + Bus：**

| 对比项     | Nacos              | Spring Cloud Config + Bus |
| ---------- | ------------------ | ------------------------- |
| 配置存储   | Nacos Server 内置  | 依赖外部 Git/数据库       |
| 实时推送   | 原生支持           | 需要 + Bus 才能推送       |
| 可视化     | 有控制台           | 无自带控制台              |
| 灰度发布   | 支持               | 不支持                    |
| 部署复杂度 | 低（单组件）       | 高（Config + Bus + MQ）   |

---

## 14、Seata 是如何解决分布式事务的？

**满分答案：**

Seata 是阿里巴巴开源的**分布式事务解决方案**，提供 AT、TCC、Saga 和 XA 四种事务模式。

**Seata 三大角色：**
- **TC（Transaction Coordinator）**：事务协调器，管理全局事务状态
- **TM（Transaction Manager）**：事务管理器，发起/提交/回滚全局事务
- **RM（Resource Manager）**：资源管理器，管理分支事务（即各微服务的本地事务）

**AT 模式（最常用，无侵入）：**
```
两阶段提交：

一阶段：
1. TM 向 TC 申请开启全局事务
2. 每个 RM 执行本地事务，同时记录 undo_log（用于回滚）
3. 本地事务提交，释放锁 → 向 TC 注册分支事务

二阶段：
- 全部成功：TC 通知各 RM 异步删除 undo_log
- 任一失败：TC 通知各 RM 根据 undo_log 回滚（反向补偿）
```

| 模式   | 侵入性 | 适用场景                         | 说明               |
| ------ | ------ | -------------------------------- | ------------------ |
| **AT**   | 无     | 大部分关系型数据库场景           | **最推荐**         |
| TCC    | 高     | 需要自定义资源锁定/释放的场景    | Try-Confirm-Cancel |
| Saga   | 中     | 长流程事务、老系统集成           | 正向补偿           |
| XA     | 无     | 数据库原生支持 XA 协议的强一致性 | 性能较差           |

**@GlobalTransactional 示例：**
```java
@GlobalTransactional
public void createOrder(Order order) {
    // 1. 本地创建订单
    orderService.create(order);
    // 2. 远程扣减库存
    stockService.deduct(order.getProductId(), order.getCount());
    // 3. 远程扣减余额
    accountService.debit(order.getUserId(), order.getAmount());
}
```

---

## 15、微服务间如何通信？Feign 和 Dubbo 的区别？

**满分答案：**

微服务间通信方式主要分为**同步通信**和**异步通信**两大类。

| 方式           | 协议                   | 场景                   |
| -------------- | ---------------------- | ---------------------- |
| HTTP/REST      | HTTP (JSON)            | 通用、跨语言           |
| RPC            | Dubbo/gRPC/Thrift      | 高性能内部服务调用     |
| 消息中间件     | RabbitMQ/Kafka/RocketMQ| 异步解耦、削峰         |

**Feign vs Dubbo：**

| 对比项       | OpenFeign (HTTP)               | Dubbo (RPC)                      |
| ------------ | ------------------------------ | -------------------------------- |
| 通信协议     | HTTP/1.1                       | 自定义协议（dubbo:// 基于TCP）   |
| 序列化       | JSON（文本，体积大）           | Hessian2/Protobuf（二进制，体积小） |
| 性能         | 较低（HTTP 开销大）            | **更高（长连接 + 二进制 + IO多路复用）** |
| 跨语言       | **天然支持**                   | 需适配（泛化调用/Protobuf）      |
| 服务治理     | 依赖 Spring Cloud 生态         | **自带完整的服务治理能力**       |
| 负载均衡     | Ribbon/LoadBalancer            | 内置多种策略 + 权重              |
| 适用场景     | 对外 API / 跨语言 / 快速开发   | 内部高性能服务调用               |

> **总结**：内部高性能服务间调用推荐 Dubbo；对外接口或需要跨语言时推荐 HTTP（Feign/RestTemplate）。

---

## 16、Sleuth + Zipkin 是如何实现链路追踪的？

**满分答案：**

**Sleuth**：Spring Cloud 的链路追踪组件，负责生成和传递 Trace ID。

**Zipkin**：分布式追踪系统，收集、存储和展示调用链数据（也可用 Skywalking、Jaeger）。

**核心概念：**

| 概念     | 说明                                               |
| -------- | -------------------------------------------------- |
| Trace    | 一次完整的请求调用链（全局唯一 Trace ID）          |
| Span     | 一次服务调用（基本工作单元），每个 Span 有自己的 ID |
| Parent   | 父 Span，形成树形调用链                            |
| Cs / Cr  | Client Sent / Client Received                      |
| Ss / Sr  | Server Sent / Server Received                      |

**工作流程：**
1. 请求进入 → 拦截器生成全局 Trace ID + Span ID
2. 调用下一个服务 → 通过 HTTP Header 传递 Trace ID 和 Parent Span ID
3. 每个服务记录自己的 Span（耗时、状态、标签）
4. 异步将 Span 数据发送到 Zipkin Server
5. Zipkin 聚合数据形成完整的调用链拓扑图

**传递的关键 HTTP Header：**
- `X-B3-TraceId`：全局追踪 ID
- `X-B3-SpanId`：当前 Span ID
- `X-B3-ParentSpanId`：父 Span ID
- `X-B3-Sampled`：是否采样

> **当前趋势**：Skywalking 由于自带的**可视化界面更丰富**、**探针无侵入**，在实际项目中越来越流行。而 Sleuth + Zipkin 的维护力度已经减弱（尤其是 Zipkin 已经基本被 Skywalking 替代）。

---

## 17、Spring Cloud Alibaba 包含哪些组件？为什么要用它？

**满分答案：**

Spring Cloud Alibaba 是阿里巴巴开源的**微服务一站式解决方案**，是 Spring Cloud 规范的具体实现。

**核心组件：**

| 组件                | 对应功能       | 替代方案（Netflix）      |
| ------------------- | -------------- | ------------------------ |
| **Nacos**           | 注册中心 + 配置中心 | Eureka + Config + Bus     |
| **Sentinel**        | 熔断降级 + 限流    | Hystrix                   |
| **Seata**           | 分布式事务         | —                         |
| **RocketMQ**        | 消息中间件         | RabbitMQ / Kafka          |
| **Dubbo**           | RPC 调用           | Feign（不是替代，是补充） |
| **Spring Cloud Alibaba Sidecar** | 异构语言接入 | —                         |

**为什么选择 Spring Cloud Alibaba：**

1. **Netflix 组件全面停更**：Eureka、Hystrix、Ribbon、Zuul 均已停止维护
2. **功能更强大**：Sentinel > Hystrix，Nacos > Eureka + Config
3. **可视化控制台**：Sentinel Dashboard、Nacos Console
4. **中文文档丰富**：阿里开源，国内社区活跃
5. **生产验证**：经过双11等极端场景验证

> **当前推荐**：新项目直接使用 Spring Cloud Alibaba 全家桶（Nacos + Sentinel + Seata + RocketMQ）。

---

## 18、Spring Cloud 和 Dubbo 如何选择？

**满分答案：**

| 对比项       | Spring Cloud                          | Dubbo                                    |
| ------------ | ------------------------------------- | ---------------------------------------- |
| 定位         | 微服务**一站式解决方案**              | **RPC 框架**（服务通信）                 |
| 通信协议     | HTTP/REST                            | 自定义 RPC 协议                          |
| 性能         | 较低（HTTP 文本协议开销）             | **更高（二进制、长连接、NIO）**          |
| 服务治理     | 提供完整的微服务治理（网关、配置、熔断等）| 需配合其他组件实现治理                  |
| 跨语言       | 天然支持                              | 需 Java 生态                             |
| 注册中心     | Eureka/Nacos/Consul 等                | Zookeeper/Nacos/Redis 等                 |
| 生态整合     | 与 Spring 生态深度整合                | Dubbo + Spring Cloud Alibaba 整合        |
| 适用场景     | 需要完整微服务治理的团队              | 追求高性能 RPC 通信的场景                |

> **最佳实践**：很多团队采用 Dubbo（服务通信）+ Nacos（注册中心 + 配置）+ Sentinel（熔断限流）的组合，既享受 Dubbo 的高性能，又有完整的治理能力。实际上就是 Spring Cloud Alibaba 的典型用法。

---

## 19、微服务下如何保证数据一致性？

**满分答案：**

微服务架构去中心化数据管理（每个服务独享数据库），数据一致性问题无法回避。

**常见的解决方案：**

| 方案               | 说明                                           | 适用场景               |
| ------------------ | ---------------------------------------------- | ---------------------- |
| **分布式事务**     | Seata AT/TCC/Saga                              | 强一致性要求           |
| **最终一致性**     | 消息队列（RocketMQ 事务消息 / RabbitMQ）       | 非实时强一致，异步解耦 |
| **补偿机制**       | 定时任务扫描异常数据 + 对账 + 人工兜底         | 资金类场景             |
| **TCC 模式**       | Try-Confirm-Cancel 两阶段提交                  | 需要资源预留的场景     |
| **事件驱动**       | 本地事务 + 事件表 + 消息队列                   | 异步解耦，最终一致性   |

**本地事务 + 事件表模式（最可靠）：**
```
1. 开启本地事务
2. 执行业务操作
3. 在同一事务中往事件表插入一条记录
4. 提交本地事务
5. 定时任务扫描事件表，发送到 MQ
6. 消费方保证幂等性消费
```

**幂等性保障方式：**
- 数据库唯一约束（unique key）
- Token 机制（请求前获取 Token，消费时校验）
- 状态机（状态流转而非直接修改）

---

## 20、如何保证微服务接口的幂等性？

**满分答案：**

幂等性是指**多次调用同一接口，结果与一次调用相同**。在分布式系统中，由于网络超时重试、消息重复消费等原因，接口幂等性是必须保证的。

**实现方案：**

| 方案                 | 说明                                               | 适用场景           |
| -------------------- | -------------------------------------------------- | ------------------ |
| **数据库唯一索引**   | 利用数据库唯一约束防止重复插入                     | 新增操作           |
| **Token 机制**       | 请求前获取 Token → 提交时校验 Token 是否已使用     | 表单提交、下单     |
| **乐观锁（版本号）** | UPDATE 时带 version 条件，只生效一次               | 更新操作           |
| **分布式锁**         | Redis/Zookeeper 分布式锁 + 业务唯一键              | 复杂业务           |
| **状态机**           | 状态不允许回退（如：待支付→已支付，不能再变回）   | 状态流转场景       |

**Token 机制示例：**
```java
// 1. 请求前获取 Token
@GetMapping("/token")
public String getToken() {
    String token = UUID.randomUUID().toString();
    redisTemplate.opsForValue().set(token, "1", 10, TimeUnit.MINUTES);
    return token;
}

// 2. 提交时携带 Token
@PostMapping("/order")
public Result createOrder(@RequestHeader("token") String token, @RequestBody Order order) {
    // 删除 Token，返回 false 说明已被使用
    Boolean flag = redisTemplate.delete(token);
    if (!flag) {
        return Result.fail("请勿重复提交");
    }
    // 执行下单逻辑
}
```

---

## 21、RestTemplate 和 WebClient 的区别？

**满分答案：**

| 对比项       | RestTemplate                     | WebClient                               |
| ------------ | -------------------------------- | --------------------------------------- |
| 编程模型     | 同步、阻塞                       | 异步、非阻塞                            |
| 底层实现     | JDK HttpURLConnection / HttpClient | Netty / Reactor-Netty                   |
| Spring 版本  | Spring 5 起标记为过时（deprecated）| Spring 5 引入，官方推荐 |
| 响应式支持   | 不支持                           | **支持（Mono / Flux）**                 |
| 性能         | 较低（阻塞线程）                 | **更高（事件驱动）**                    |
| 适用场景     | 低频调用、简单场景               | **高并发、流式处理**                    |

**WebClient 示例：**
```java
WebClient client = WebClient.create("http://user-service");

Mono<User> user = client.get()
    .uri("/user/{id}", 1)
    .retrieve()
    .bodyToMono(User.class);

// 非阻塞，链式调用
user.subscribe(u -> System.out.println(u.getName()));
```

---

## 22、Spring Cloud Gateway 的限流怎么实现？

**满分答案：**

Spring Cloud Gateway 提供了**RequestRateLimiterGatewayFilterFactory**，基于 **Token Bucket（令牌桶）算法** 实现限流，默认配合 Redis 使用。

**实现步骤：**
1. 引入 `spring-boot-starter-data-redis-reactive` 依赖
2. 配置 KeyResolver（定义限流维度）
3. 配置路由限流参数

**示例配置：**
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://user-service
          predicates:
            - Path=/api/user/**
          filters:
            - name: RequestRateLimiter
              args:
                key-resolver: "#{@ipKeyResolver}"
                redis-rate-limiter.replenishRate: 10    # 每秒令牌投放数
                redis-rate-limiter.burstCapacity: 20    # 最大令牌容量
```

**限流维度（KeyResolver）：**
- 按 IP 限流（最常见）
- 按用户 ID 限流
- 按接口路径限流
- 组合维度限流

---

## 23、Spring Cloud 各版本对应关系？

**满分答案：**

Spring Cloud 使用 **Release Train**（发布列车）命名，版本号与 Spring Boot 有严格的对应关系。

| Spring Cloud 版本 | Spring Boot 版本 | 状态     | 说明                         |
| ----------------- | ---------------- | -------- | ---------------------------- |
| Hoxton.SR12       | 2.2.x / 2.3.x    | 已停更   | Netflix 组件最后版本         |
| 2020.0.x (Ilford) | 2.4.x / 2.5.x    | 已停更   | Ribbon → LoadBalancer 过渡   |
| 2021.0.x (Jubilee)| 2.6.x / 2.7.x    | 维护中   | Spring Boot 2.7 对应的稳定版 |
| 2022.0.x (Kilburn)| 3.0.x            | 维护中   | JDK 17 + Spring Boot 3.x     |
| 2023.0.x (Leyton) | 3.1.x / 3.2.x    | 活跃维护 | 当前主流                     |
| 2024.0.x          | 3.3.x / 3.4.x    | 活跃维护 | 最新版本                     |

> **版本选择建议**：生产环境优先选择与当前 Spring Boot 版本匹配的**最新稳定版**。

---

## 24、如何设计一个高可用的注册中心？

**满分答案：**

高可用注册中心的设计需要考虑以下几个关键维度：

**1. 集群部署**
- 注册中心本身以**集群模式**部署（至少3个节点）
- Eureka：Peer to Peer，节点间相互注册
- Nacos：Raft 协议保证集群一致性

**2. 服务消费者本地缓存**
- 消费者本地**缓存服务地址列表**
- 注册中心故障时仍可通过本地缓存调用服务
- Nacos 使用本地快照（snapshot）文件持久化

**3. 健康检查**
- 主动检查（注册中心定期 PING 服务实例）
- 被动检查（服务实例定时发送心跳）
- 不健康实例**及时下线**

**4. 故障转移**
- 多个注册中心地址配置
- Eureka：自我保护机制
- Nacos：TCP/HTTP/MySQL 多种健康检查

**5. 数据持久化**
- Eureka：内存 + 增量备份，不推荐持久化
- Nacos：MySQL 存储（集群模式），嵌入式 Derby（单机模式）

---

## 25、你们项目中用了哪些 Spring Cloud 组件？架构是什么样的？

**满分答案（参考话术）：**

> 面试官您好，我们项目目前使用的是 **Spring Cloud Alibaba 全家桶**：
>
> - **Nacos** 作为**注册中心和配置中心**（替代已停止维护的 Eureka + Config）
> - **Sentinel** 负责**熔断降级和限流**（相比 Hystrix 更强大，有可视化控制台）
> - **Spring Cloud Gateway** 作为**API 网关**，统一鉴权、路由分发
> - **OpenFeign** 进行**服务间的声明式 HTTP 调用**，内部集成 LoadBalancer 实现负载均衡
> - **Seata** 的 **AT 模式**解决**分布式事务**（例如订单+库存+账户的跨服务事务）
> - **Sleuth + Skywalking** 做**链路追踪**和性能监控
> - **RocketMQ** 用于**异步解耦**（如订单完成后发送通知、生成报表等）
>
> **架构拓扑：**
> ```
> 客户端 → Nginx → Spring Cloud Gateway → 各微服务集群
>                                           ↓
>                             Nacos（注册发现 + 配置中心）
>                             Sentinel（熔断限流）
>                             Seata（分布式事务）
>                             Skywalking（链路追踪）
>                             RocketMQ（异步消息）
> ```

---

## 26、微服务如何实现灰度发布？

**满分答案：**

灰度发布（金丝雀发布）是指在**小部分用户**上先体验新版本，验证没问题后再全量发布。

**实现方案：**

| 方案               | 说明                                           |
| ------------------ | ---------------------------------------------- |
| **Nacos 权重**     | 调整服务实例权重，逐步引流                   |
| **Gateway 路由**   | 基于请求头/参数路由到灰度版本                 |
| **Nacos 元数据**   | 通过 metadata 标记版本，结合负载均衡策略       |
| **Istio/Service Mesh** | 流量染色 + 路由到指定版本                 |

**Gateway + Nacos 灰度方案示例：**
```yaml
# 通过请求头 version=v2 路由到灰度服务
spring:
  cloud:
    gateway:
      routes:
        - id: user-service-gray
          uri: lb://user-service
          predicates:
            - Path=/api/user/**
            - Header=version, v2
          filters:
            - AddRequestHeader=X-Version, v2
```

---

## 27、分布式系统中 Session 如何管理？

**满分答案：**

由于微服务是无状态的，Session 管理需要通过外部存储来共享。

| 方案                      | 说明                                           | 优缺点                         |
| ------------------------- | ---------------------------------------------- | ------------------------------ |
| **Token/JWT**             | 客户端存储 Token，服务端无状态                 | ✅无状态 ❌Token 无法主动失效 |
| **Redis 集中存储**        | Spring Session + Redis，多服务共享             | ✅可主动失效 ❌需额外基础设施  |
| **网关统一鉴权**          | Gateway 层验证 JWT 并解析用户信息传给下游      | ✅无状态，推荐                 |
| **SSO 单点登录**          | OAuth2 + JWT，统一认证中心                     | ✅适合大型系统                 |

**推荐方案（Gateway + JWT）：**
```
1. 用户登录 → 认证服务返回 JWT Token
2. 客户端每次请求携带 Token 在 Header 中
3. Gateway 统一校验 JWT
4. Gateway 将解析出的用户信息以 Header 形式传递给下游微服务
5. 微服务无需重复认证，直接从 Header 获取用户信息
```

---

[//]: # "以上内容涵盖了 Spring Cloud 面试中最常见的 ~27 个核心问题，包含分布式系统理论、核心组件对比以及实际项目经验。"
