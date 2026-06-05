# Spring 面试题

## 1、什么是 Spring？

**满分答案：**
Spring 是一个**轻量级的开源Java开发框架**，诞生于2003年，旨在**简化企业级应用开发**。它最核心的两大特性是：
- **IoC（控制反转）**：将对象的创建、组装、管理权交给Spring容器，实现解耦
- **AOP（面向切面编程）**：将日志、事务等通用功能与业务逻辑分离，提高复用性

Spring 全家桶还包括 Spring Boot、Spring Cloud、Spring Data、Spring Security 等生态组件。

---

## 2、你们项目为什么使用 Spring 框架？

**满分答案：**
1. **轻量级**：启动快、依赖少、对代码无侵入
2. **IoC 解耦**：对象由容器管理，代码之间不再直接 new，便于测试和维护
3. **AOP 支持**：统一处理日志、权限、事务等横切逻辑
4. **一站式生态**：Web MVC、JDBC、事务、Security、Boot、Cloud 等
5. **社区活跃**：文档丰富、问题容易找到解决方案、版本持续迭代
6. **与主流框架集成方便**：如 MyBatis、Hibernate、Thymeleaf 等

---

## 3、Autowired 和 Resource 关键字的区别？

**满分答案：**

| 对比项         | @Autowired                  | @Resource                             |
| -------------- | --------------------------- | ------------------------------------- |
| 来源           | Spring 框架                 | JDK 标准注解（javax.annotation）      |
| 默认装配方式   | **byType**（按类型）        | **byName**（按名称）                  |
| 指定名称的方式 | 配合 @Qualifier("beanName") | name 属性：@Resource(name="beanName") |
| 是否必须       | required 属性控制           | 默认必须                              |
| 适用范围       | 字段、构造器、setter        | 字段、setter                          |

**示例：**
```java
@Autowired
@Qualifier("userService")
private UserService userService;

@Resource(name = "userService")
private UserService userService;
```

---

## 4、依赖注入的方式有哪些？

**满分答案：**
| 注入方式    | 说明                                            | 示例                                    |
| ----------- | ----------------------------------------------- | --------------------------------------- |
| 构造器注入  | 通过构造方法传入依赖，**推荐用于必须依赖**      | `public UserService(UserDao dao){}`     |
| Setter 注入 | 通过 set 方法注入，**可选依赖**                 | `public void setUserDao(UserDao dao){}` |
| 字段注入    | 直接在字段上用 @Autowired，**最简洁但不易测试** | `@Autowired private UserDao dao;`       |
| 接口注入    | Spring 已废弃，不推荐                           | —                                       |

> 官方推荐：**构造器注入**（不可变、更安全、方便单元测试）。

---

## 5、Spring 有哪些核心的模块？

**满分答案：**

| 模块              | 作用                                             |
| ----------------- | ------------------------------------------------ |
| spring-core       | 核心工具类 + IoC 基础                            |
| spring-beans      | BeanFactory 和 Bean 管理                         |
| spring-context    | ApplicationContext，提供国际化、事件、资源访问等 |
| spring-aop        | 面向切面编程支持                                 |
| spring-expression | SpEL 表达式语言                                  |
| spring-jdbc       | JDBC 模板与数据源支持                            |
| spring-tx         | 声明式事务管理                                   |
| spring-web        | Web 应用基础，含 Multipart、Servlet 等           |
| spring-webmvc     | Spring MVC 框架实现                              |
| spring-orm        | 与 Hibernate、JPA 等 ORM 框架集成                |

---

## 6、说说你对 Spring MVC 的理解？

**满分答案：**

Spring MVC 是基于 **Model-View-Controller** 设计模式的 Web 框架，核心组件是 `DispatcherServlet`。

**完整执行流程：**
1. 用户请求 → `DispatcherServlet`
2. `DispatcherServlet` → `HandlerMapping`（找能处理该 URL 的 Controller）
3. `HandlerMapping` 返回 `HandlerExecutionChain`（Handler + Interceptors）
4. `DispatcherServlet` → `HandlerAdapter`（找到能执行该 Handler 的适配器）
5. `HandlerAdapter` 执行 `Controller` 业务逻辑
6. `Controller` 返回 `ModelAndView`
7. `HandlerAdapter` 将 `ModelAndView` 返回给 `DispatcherServlet`
8. `DispatcherServlet` → `ViewResolver`（解析逻辑视图名为真实视图）
9. `ViewResolver` 返回 `View` 对象
10. `DispatcherServlet` 将 Model 数据填充到 View 中渲染 HTML
11. 最终响应返回给客户端

**核心组件说明：**
- `DispatcherServlet`：前端控制器，中央调度器
- `HandlerMapping`：URL → Handler 映射器
- `HandlerAdapter`：执行 Handler 的适配器
- `Controller`：业务处理器
- `ViewResolver`：视图解析器
- `View`：视图渲染（JSP / JSON / Thymeleaf 等）

---

## 7、Spring MVC 的常用注解？

**满分答案：**

| 注解                                                         | 作用                                           |
| ------------------------------------------------------------ | ---------------------------------------------- |
| `@Controller`                                                | 标记该类为 Spring MVC 控制器                   |
| `@RestController`                                            | `@Controller` + `@ResponseBody`，适合 REST API |
| `@RequestMapping`                                            | 类/方法级别 URL 映射                           |
| `@GetMapping` / `@PostMapping` / `@PutMapping` / `@DeleteMapping` | 快捷映射                                       |
| `@RequestParam`                                              | 获取请求参数                                   |
| `@PathVariable`                                              | 获取路径参数，如 `/user/{id}`                  |
| `@RequestBody`                                               | JSON/XML → Java 对象                           |
| `@ResponseBody`                                              | Java 对象 → JSON/XML                           |
| `@ModelAttribute`                                            | 绑定表单/请求参数到对象                        |
| `@ExceptionHandler`                                          | 局部异常处理                                   |
| `@ControllerAdvice`                                          | 全局异常/数据绑定增强                          |

---

## 8、你对 Spring 的 AOP 是如何理解的？

**满分答案：**

AOP（面向切面编程）将**横切关注点**（日志、事务、权限）与**核心业务逻辑**分离。

**核心概念：**
- **切面（Aspect）**：横切关注点的模块化，如 `LogAspect`
- **连接点（JoinPoint）**：可被增强的方法执行点
- **通知（Advice）**：切面在连接点上执行的动作（前置、后置、环绕等）
- **切入点（Pointcut）**：匹配连接点的表达式
- **织入（Weaving）**：将切面应用到目标对象的过程

**Spring AOP 底层原理：**
- 目标对象**有接口** → JDK 动态代理（`Proxy.newProxyInstance`）
- 目标对象**无接口** → CGLIB 字节码生成子类代理

---

## 9、Spring AOP 和 AspectJ AOP 的区别？

**满分答案：**

| 对比项             | Spring AOP             | AspectJ                      |
| ------------------ | ---------------------- | ---------------------------- |
| 织入时机           | **运行时**             | 编译时 / 类加载时 / 运行时   |
| 实现方式           | JDK 动态代理 / CGLIB   | 字节码修改（AspectJ 编译器） |
| 是否需独立编译器   | 否                     | 是（ajc）                    |
| 是否支持字段拦截   | 否                     | 是                           |
| 是否支持构造器拦截 | 否                     | 是                           |
| 性能               | 较低（运行时生成代理） | 较高（直接执行织入代码）     |
| 复杂切面支持       | 有限                   | 完全支持                     |

> Spring AOP 适合 80% 企业场景，复杂需求可集成 AspectJ。

---

## 10、Spring AOP 中关注点和切入点是什么？

**满分答案：**
- **关注点（Concern）**：分为**核心关注点**（业务逻辑）和**横切关注点**（日志、事务、安全）。AOP 将横切关注点模块化为 **切面**。
- **切入点（Pointcut）**：通过**表达式**定义在**哪些连接点**上应用通知。例如：
  ```java
  @Pointcut("execution(* com.service.*.*(..))")
  ```

---

## 11、什么是通知，有哪些类型？

**满分答案：**

通知（Advice）是切面在**特定连接点**执行的动作。Spring 支持 **5 种通知**：

| 通知类型     | 注解              | 执行时机                                                     |
| ------------ | ----------------- | ------------------------------------------------------------ |
| 前置通知     | `@Before`         | 目标方法执行**之前**                                         |
| 后置通知     | `@After`          | 目标方法执行**之后**（无论是否异常）                         |
| 后置返回通知 | `@AfterReturning` | 目标方法**正常返回**后                                       |
| 后置异常通知 | `@AfterThrowing`  | 目标方法**抛出异常**后                                       |
| 环绕通知     | `@Around`         | 目标方法执行**前后都可以自定义逻辑**，可控制是否执行目标方法 |

> 环绕通知功能最强，可以完全控制目标方法的调用。

---

## 12、你对 Spring IoC 是如何理解的？

**满分答案：**

IoC（控制反转）将**对象的创建、管理、装配权**从程序员手中**反转**给 Spring 容器。

**核心概念：**
- **Bean**：由 Spring IoC 容器管理的对象
- **BeanFactory / ApplicationContext**：IoC 容器接口
- **DI（依赖注入）**：IoC 的实现方式，容器自动将依赖对象注入到需要的地方

**好处：**
- 降低组件之间的耦合度
- 方便单元测试（可注入 Mock 对象）
- 集中管理 Bean 生命周期（单例、作用域、初始化销毁）

**示例对比：**
```java
// 传统方式：紧耦合
UserService service = new UserServiceImpl();

// Spring 方式：解耦
@Autowired
private UserService service;
```

---

## 13、Spring Bean 的生命周期？

**满分答案：**

完整生命周期分为 **4 大阶段**：

```
实例化 → 属性填充 → 初始化 → 销毁
```

**详细 10 个步骤：**
1. **实例化**：通过构造器或工厂方法创建 Bean 实例
2. **属性填充**：依赖注入（setter/字段）
3. **Aware 接口回调**：
   - `BeanNameAware` → 设置 beanName
   - `BeanClassLoaderAware`
   - `BeanFactoryAware` → 传入 BeanFactory
4. **BeanPostProcessor.beforeInitialization**
5. **@PostConstruct** 或 `InitializingBean.afterPropertiesSet`
6. **自定义 init-method**
7. **BeanPostProcessor.afterInitialization**
8. **Bean 就绪**（此时可用）
9. **容器销毁** → @PreDestroy
10. `DisposableBean.destroy()` 或 **自定义 destroy-method**

> 最常考：第 4~7 步（初始化前后 + InitializingBean + init-method 的顺序）。

---

## 14、Spring 支持的几种 Bean 作用域？

**满分答案：**

| 作用域            | 说明                                            |
| ----------------- | ----------------------------------------------- |
| singleton（默认） | 容器中只有一个 Bean 实例，单例                  |
| prototype         | 每次请求/获取都创建新实例                       |
| request           | 每个 HTTP 请求创建一个实例（仅 Web 环境）       |
| session           | 每个 HTTP Session 创建一个实例（仅 Web 环境）   |
| application       | 每个 ServletContext 创建一个实例（仅 Web 环境） |
| websocket         | 每个 WebSocket 连接创建一个实例（仅 Web 环境）  |

> 面试重点：singleton 与 prototype 的区别，以及 request/session 的作用域。

---

## 15、Spring 基于 xml 注入 Bean 的方式？

**满分答案：**

| 注入方式     | XML 配置                                                     |
| ------------ | ------------------------------------------------------------ |
| 构造器注入   | `<constructor-arg ref="userDao"/>` 或 `<constructor-arg value="张三"/>` |
| Setter 注入  | `<property name="userDao" ref="userDao"/>`                   |
| p 命名空间   | `<bean p:userDao-ref="userDao"/>`                            |
| 静态工厂方法 | `<bean factory-method="getInstance"/>`                       |
| 实例工厂方法 | `<bean factory-bean="factoryBean" factory-method="create"/>` |

---

## 16、Spring 用了哪些设计模式？

**满分答案：**

| 设计模式       | Spring 中的体现                                              |
| -------------- | ------------------------------------------------------------ |
| **单例模式**   | Bean 默认作用域（singleton）                                 |
| **工厂模式**   | BeanFactory、FactoryBean                                     |
| **原型模式**   | prototype 作用域                                             |
| **代理模式**   | AOP 动态代理（JDK / CGLIB）                                  |
| **模板方法**   | JdbcTemplate、RestTemplate、RedisTemplate                    |
| **策略模式**   | InstantiationStrategy、HandlerMapping 等                     |
| **观察者模式** | ApplicationEvent + ApplicationListener                       |
| **适配器模式** | HandlerAdapter（适配不同 Controller）                        |
| **装饰者模式** | BeanWrapper、HttpServletRequestWrapper                       |
| **责任链模式** | BeanPostProcessor 链、Interceptor 链、SpringSecurity 过滤器链 |
| **委托模式**   | BeanDefinitionParserDelegate                                 |
| **组合模式**   | CompositeCacheManager、CompositeFilter                       |

> 能答出 5~6 个 + 具体例子即可满分。

---

## 17、Spring 是如何解决循环依赖的？

**满分答案：**

Spring 通过 **三级缓存** 解决**单例 Bean 的字段注入 / setter 注入**循环依赖。

**三级缓存结构：**
- 一级缓存 `singletonObjects`：已完全初始化的 Bean 实例
- 二级缓存 `earlySingletonObjects`：已实例化但未填充属性的 Bean 实例（提前暴露）
- 三级缓存 `singletonFactories`：存放 Bean 的 ObjectFactory（用于提前生成代理对象）

**解决过程（以 A → B → A 为例）：**
1. 实例化 A → 将 A 的 `ObjectFactory` 放入**三级缓存**
2. 填充 A 属性时发现需要 B → 去获取 B
3. 实例化 B → 将 B 的 `ObjectFactory` 放入**三级缓存**
4. 填充 B 属性时发现需要 A → 去获取 A
5. 从**三级缓存**拿到 A 的 ObjectFactory → 生成 A 的早期引用 → 放入**二级缓存** → 删除三级缓存
6. B 拿到 A → B 完成初始化 → B 放入**一级缓存**
7. A 拿到 B → A 完成初始化 → A 放入**一级缓存**

**局限性（必须知道）：**
- ✅ 只能解决 **单例 + 字段/setter 注入** 的循环依赖
- ❌ 无法解决 **构造器注入** 的循环依赖
- ❌ 无法解决 **prototype 作用域** 的循环依赖

> 能说出“三级缓存” + “为什么需要三级缓存（处理AOP代理）”就是大神级答案。

---

## 18、事务的隔离级别？

**满分答案：**

| 隔离级别         | 脏读 | 不可重复读 | 幻读 | 说明                            |
| ---------------- | ---- | ---------- | ---- | ------------------------------- |
| READ_UNCOMMITTED | ✅    | ✅          | ✅    | 最低隔离，性能最高              |
| READ_COMMITTED   | ❌    | ✅          | ✅    | Oracle 默认，避免脏读           |
| REPEATABLE_READ  | ❌    | ❌          | ✅    | MySQL 默认，避免脏读+不可重复读 |
| SERIALIZABLE     | ❌    | ❌          | ❌    | 最高隔离，表级锁，性能最差      |

> 隔离级别从低到高：读未提交 → 读已提交 → 可重复读 → 串行化。

---

## 19、事务的7种传播级别？

**满分答案：**

| 传播级别           | 含义                             |
| ------------------ | -------------------------------- |
| `REQUIRED`（默认） | 当前有事务则加入，否则新建       |
| `SUPPORTS`         | 当前有事务则加入，否则非事务执行 |
| `MANDATORY`        | 当前必须有事务，否则抛异常       |
| `REQUIRES_NEW`     | **总是新建事务**，挂起当前事务   |
| `NOT_SUPPORTED`    | 非事务执行，有则挂起             |
| `NEVER`            | 非事务执行，有则抛异常           |
| `NESTED`           | 嵌套事务（基于 Savepoint）       |

> 最常考：`REQUIRED` 与 `REQUIRES_NEW` 的区别。

---

## 20、Spring 事务的实现方式？

**满分答案：**
- **编程式事务**：
  - 手动调用 `TransactionTemplate.execute()`
  - 或直接使用 `PlatformTransactionManager`
- **声明式事务**（推荐）：
  - 基于 `@Transactional` 注解
  - 或基于 XML 配置 `<tx:advice>`

> 底层原理：AOP + 动态代理，在代理类中开启/提交/回滚事务。

---

## 21、事务的三要素是什么？

**满分答案：**

Spring 事务管理的核心三要素：
1. **数据源（DataSource）**：数据库连接来源
2. **事务管理器（TransactionManager）**：
   - `DataSourceTransactionManager`（JDBC/MyBatis）
   - `JpaTransactionManager`（JPA/Hibernate）
   - `JtaTransactionManager`（分布式事务）
3. **事务属性（TransactionAttribute）**：
   - 隔离级别
   - 传播行为
   - 超时时间
   - 只读标志
   - 异常回滚规则（rollbackFor/noRollbackFor）

> 也有人用“隔离级别 + 传播行为 + 超时时间”作为三要素，两种说法均正确。

