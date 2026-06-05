# Spring Boot 面试题

## 1、为什么要用 Spring Boot？

**满分答案：**

Spring Boot 是基于 Spring 框架的**快速开发脚手架**，主要解决传统 Spring 开发中的以下痛点：

| 传统 Spring 痛点      | Spring Boot 解决方案                           |
| --------------------- | ---------------------------------------------- |
| 大量 XML / Java 配置  | **约定大于配置**，自动配置                     |
| 依赖版本管理困难      | **起步依赖（Starter）**，统一版本管理          |
| 整合第三方框架繁琐    | 官方提供 Starter，一键引入                     |
| 需要手动部署到 Tomcat | **内置 Web 容器**（Tomcat / Jetty / Undertow） |
| 缺少监控和管理        | Actuator 提供生产级监控                        |

**核心优势：**
1. 快速创建独立可运行的 Spring 应用
2. 自动配置，减少样板代码
3. 生产就绪（健康检查、指标监控）
4. 与微服务生态（Spring Cloud）无缝集成

---

## 2、Spring Boot 的核心注解是哪个？它主要由哪几个注解组成？

**满分答案：**

核心注解：**`@SpringBootApplication`**

`@SpringBootApplication` 是一个**组合注解**，主要由以下 3 个注解组成：

| 注解                       | 作用                                                         |
| -------------------------- | ------------------------------------------------------------ |
| `@SpringBootConfiguration` | 标明该类为 Spring Boot 配置类（底层是 `@Configuration`）     |
| `@EnableAutoConfiguration` | **核心**：开启自动配置，根据 classpath 依赖自动注册 Bean     |
| `@ComponentScan`           | 扫描当前包及子包下的 `@Component`、`@Service`、`@Controller` 等 |

> 补充：`@SpringBootApplication` 也支持自定义 `scanBasePackages` 参数来指定扫描路径。

---

## 3、Spring Boot 的自动配置原理？

**满分答案：**

自动配置是 Spring Boot 的**最核心特性**，通过 `@EnableAutoConfiguration` 实现。

**工作流程：**
1. `@SpringBootApplication` → `@EnableAutoConfiguration` → `@Import(AutoConfigurationImportSelector.class)`
2. `AutoConfigurationImportSelector` 读取 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`（Spring Boot 3.x）或 `META-INF/spring.factories`（Spring Boot 2.x）中注册的自动配置类
3. 每个自动配置类使用 `@ConditionalOnXxx` 条件注解判断是否生效
4. 满足条件 → 自动创建对应的 Bean

**关键条件注解：**

| 注解                        | 作用                                     |
| --------------------------- | ---------------------------------------- |
| `@ConditionalOnClass`       | classpath 存在指定类时生效               |
| `@ConditionalOnMissingClass`| classpath 不存在指定类时生效             |
| `@ConditionalOnBean`        | 容器中存在指定 Bean 时生效               |
| `@ConditionalOnMissingBean` | 容器中不存在指定 Bean 时生效             |
| `@ConditionalOnProperty`    | 配置文件中存在指定属性时生效             |
| `@ConditionalOnResource`    | classpath 存在指定资源时生效             |
| `@ConditionalOnWebApplication` | 当前是 Web 应用时生效                 |

**示例（Spring Boot 的 DataSource 自动配置逻辑）：**
```java
@Configuration
@ConditionalOnClass({DataSource.class, EmbeddedDatabaseType.class})
@ConditionalOnMissingBean(DataSource.class)  // 用户自定义了 DataSource 则不生效
public class DataSourceAutoConfiguration {
    // 自动创建 DataSource Bean
}
```

> 能清楚解释 "自动配置类 + 条件注解 + spring.factories" 三角关系即可满分。

---

## 4、如何自定义 Starter？

**满分答案：**

自定义 Starter 三步走：

**第 1 步：创建配置属性类**
```java
@ConfigurationProperties(prefix = "my-starter")
public class MyStarterProperties {
    private String name = "default";
    private boolean enabled = true;
    // getter / setter
}
```

**第 2 步：创建自动配置类**
```java
@Configuration
@ConditionalOnProperty(prefix = "my-starter", name = "enabled", havingValue = "true", matchIfMissing = true)
@EnableConfigurationProperties(MyStarterProperties.class)
public class MyStarterAutoConfiguration {
    @Bean
    @ConditionalOnMissingBean
    public MyService myService(MyStarterProperties properties) {
        return new MyService(properties.getName());
    }
}
```

**第 3 步：注册自动配置类**

在 `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`（Spring Boot 3.x）中写入：
```
com.example.MyStarterAutoConfiguration
```

> 命名规范：官方 Starter 命名为 `spring-boot-starter-xxx`，第三方 Starter 命名为 `xxx-spring-boot-starter`。

---

## 5、Spring Boot 的启动方式？

**满分答案：**

| 启动方式                | 说明                                               |
| ----------------------- | -------------------------------------------------- |
| **主类直接运行**        | IDE 中运行 `main` 方法（开发最常用）               |
| **Maven / Gradle 插件** | `mvn spring-boot:run` 或 `gradle bootRun`          |
| **打成 JAR 包运行**     | `java -jar xxx.jar`（生产常用）                    |
| **打成 WAR 包部署**     | 部署到外部 Tomcat / Jetty 等容器（需排除内置容器） |

**JAR 包启动示例：**
```bash
java -jar myapp.jar --server.port=8081
```

**WAR 包部署注意事项：**
- `packaging` 改为 `war`
- 排除内置 Tomcat 依赖
- 主类继承 `SpringBootServletInitializer`

---

## 6、如何理解 Spring Boot 里面的 Starter？

**满分答案：**

Starter 是 Spring Boot 的 **一站式依赖描述符**，它是一个 **Maven/Gradle 依赖**，里面封装了：

- 功能所需的所有 **依赖 JAR**（已版本兼容）
- **自动配置类**（在 `AutoConfiguration.imports` 中注册）
- 可选的 **默认配置属性**

**核心思想：**
> 开发者只需要**引入一个 Starter**，就能获得某个功能的完整集成，无需关心依赖版本和配置细节。

**工作原理：**
1. 引入 `spring-boot-starter-web`
2. Spring Boot 检测到 classpath 中存在 `Tomcat`、`Spring MVC`
3. `@EnableAutoConfiguration` 自动注册 `DispatcherServlet`、`Tomcat` 等 Bean
4. 开发者只需在 `application.yml` 中覆盖配置即可

---

## 7、如何在 Spring Boot 启动的时候执行一些特定的代码？

**满分答案：**

Spring Boot 提供了 **5 种** 在启动时执行特定代码的方式：

| 方式                                          | 说明                                                        | 使用场景                         |
| --------------------------------------------- | ----------------------------------------------------------- | -------------------------------- |
| `@PostConstruct`                              | Bean 初始化后执行                                           | 简单初始化逻辑                   |
| `CommandLineRunner`                           | 容器启动后执行，可访问命令行参数                            | 初始化数据、预热缓存             |
| `ApplicationRunner`                           | 同 `CommandLineRunner`，但参数封装为 `ApplicationArguments` | 更灵活的参数解析                 |
| `InitializingBean.afterPropertiesSet`         | Bean 属性填充完成后执行                                     | 更底层的初始化                   |
| `@EventListener(ApplicationReadyEvent.class)` | 监听应用就绪事件                                            | 需要在所有 Bean 完全初始化后执行 |

**示例代码：**
```java
@Component
public class MyRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("容器启动完成，执行初始化逻辑");
    }
}
```

**执行顺序：**
`@PostConstruct` → `InitializingBean` → `CommandLineRunner` / `ApplicationRunner` → `ApplicationReadyEvent`

> 可以使用 `@Order` 注解控制 `CommandLineRunner` 的执行顺序。

---

## 8、Spring Boot 需要独立的容器运行么？

**满分答案：**

**不需要。**

Spring Boot 通过 `spring-boot-starter-web` **内置了 Web 容器**：

| 内置容器 | 说明               |
| -------- | ------------------ |
| Tomcat   | 默认，最常用       |
| Jetty    | 可替换 Tomcat      |
| Undertow | 高性能、内存占用低 |

**运行方式：**
- 开发环境：直接运行 `main` 方法
- 生产环境：`java -jar xxx.jar`

**替换内置容器示例：**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

---

## 9、Spring Boot 的监视器是什么？

**满分答案：**

监视器是 **Spring Boot Actuator**，通过引入 `spring-boot-starter-actuator` 依赖获得。

**主要功能：**
- 监控应用健康状态
- 查看应用指标（内存、线程、HTTP 请求等）
- 管理应用（动态修改日志级别、关闭应用等）
- 暴露 `/actuator` 端点

**常用端点：**

| 端点                 | 作用                      |
| -------------------- | ------------------------- |
| `/actuator/health`   | 应用健康状态（UP / DOWN） |
| `/actuator/info`     | 应用自定义信息            |
| `/actuator/metrics`  | 应用指标数据              |
| `/actuator/env`      | 环境配置信息              |
| `/actuator/loggers`  | 查看/修改日志级别         |
| `/actuator/shutdown` | 优雅关闭应用（需开启）    |
| `/actuator/beans`    | 查看容器中所有 Bean       |
| `/actuator/mappings` | 查看所有 URL 映射         |

**安全配置示例：**
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics  # 暴露指定端点
  endpoint:
    health:
      show-details: always
    shutdown:
      enabled: true
```

---

## 10、如何使用 Spring Boot 实现异常处理？

**满分答案：**

Spring Boot 通过 **`@ControllerAdvice` + `@ExceptionHandler`** 实现**全局统一异常处理**。

**完整示例：**
```java
@RestControllerAdvice  // = @ControllerAdvice + @ResponseBody
public class GlobalExceptionHandler {

    // 处理自定义业务异常
    @ExceptionHandler(BusinessException.class)
    public Result handleBusinessException(BusinessException e) {
        return Result.error(e.getCode(), e.getMessage());
    }

    // 处理参数校验异常
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public Result handleValidException(MethodArgumentNotValidException e) {
        String message = e.getBindingResult()
            .getAllErrors().get(0).getDefaultMessage();
        return Result.error(400, message);
    }

    // 兜底处理所有未捕获异常
    @ExceptionHandler(Exception.class)
    public Result handleException(Exception e) {
        log.error("系统异常", e);
        return Result.error(500, "服务器内部错误");
    }
}
```

**补充方式：**
- `ErrorController`：自定义 Spring Boot 默认错误页面
- `@ResponseStatus`：为指定异常指定 HTTP 状态码

---

## 11、Spring Boot 常用的一些 Starter？

**满分答案：**

| Starter                            | 作用                               |
| ---------------------------------- | ---------------------------------- |
| `spring-boot-starter-web`          | Web 开发（含 Tomcat + Spring MVC） |
| `spring-boot-starter-data-jpa`     | JPA / Hibernate 数据访问           |
| `spring-boot-starter-data-redis`   | Redis 集成                         |
| `spring-boot-starter-data-mongodb` | MongoDB 集成                       |
| `spring-boot-starter-jdbc`         | JDBC 模板 + HikariCP 连接池        |
| `spring-boot-starter-amqp`         | RabbitMQ 消息队列                  |
| `spring-boot-starter-kafka`        | Kafka 消息队列                     |
| `spring-boot-starter-security`     | Spring Security 安全框架           |
| `spring-boot-starter-mail`         | 邮件发送                           |
| `spring-boot-starter-test`         | 单元测试（JUnit + Mockito）        |
| `spring-boot-starter-actuator`     | 生产监控                           |
| `mybatis-spring-boot-starter`      | MyBatis 集成（第三方）             |

> 能说出 8 个以上 + 用途即可满分。

---

## 12、Spring Boot 的常用热部署方式是什么？

**满分答案：**

Spring Boot 热部署主要有以下方式：

| 方式                     | 说明                             | 推荐度     |
| ------------------------ | -------------------------------- | ---------- |
| **spring-boot-devtools** | 官方提供的热部署工具             | ✅ 开发常用 |
| **JRebel**               | 第三方商业化热部署工具，功能最强 | 付费       |
| **手动重启**             | 修改代码后手动 `Ctrl + F5`       | ❌ 不推荐   |
| **Spring Loaded**        | 旧版，已不推荐                   | ❌          |

**Devtools 配置：**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

**Devtools 工作原理：**
- 监控 classpath 下文件变化
- 检测到变化后**自动重启应用**（通过两个 ClassLoader 实现）
- `restart` 比 `reload` 快，但仍不是真正的"热替换"
- 生产环境自动禁用

**配置触发重启的路径：**
```yaml
spring:
  devtools:
    restart:
      additional-paths: src/main/java  # 额外监控路径
      exclude: static/**, public/**    # 排除静态资源
```

---

## 13、如何理解 Spring Boot 的配置加载顺序？

**满分答案：**

Spring Boot 配置加载具有**优先级**，高优先级配置会覆盖低优先级配置。

**从高到低优先级（序号越小优先级越高）：**

| 优先级 | 配置来源                               | 说明                                      |
| ------ | -------------------------------------- | ----------------------------------------- |
| 1      | 命令行参数                             | `java -jar app.jar --server.port=8081`    |
| 2      | 系统环境变量                           | `SPRING_APPLICATION_JSON` 等              |
| 3      | JVM 系统属性                           | `-Dserver.port=8081`                      |
| 4      | `@PropertySource`                      | 注解导入的配置文件                        |
| 5      | `application-{profile}.properties/yml` | 多环境配置文件                            |
| 6      | `application.properties/yml`           | 默认配置文件                              |
| 7      | Jar 包内的默认配置                     | 来自依赖 Starter 的 `application-xxx.yml` |

**示例：**
```bash
# 命令行参数优先级最高，会覆盖配置文件中的 port
java -jar app.jar --server.port=8081
```

> 理解这个顺序对排查配置不生效问题至关重要。

---

## 14、Spring Boot 的核心配置文件有哪些，他们的区别？

**满分答案：**

Spring Boot 支持两种格式的配置文件：

| 配置文件                                     | 加载时机             | 用途                                             |
| -------------------------------------------- | -------------------- | ------------------------------------------------ |
| `application.yml` / `application.properties` | 应用启动**早期**     | 主配置文件，存储所有业务配置                     |
| `bootstrap.yml` / `bootstrap.properties`     | **更早**（引导阶段） | Spring Cloud 配置中心、加密/解密、上下文初始配置 |

**核心区别：**

| 对比项   | application.yml | bootstrap.yml                       |
| -------- | --------------- | ----------------------------------- |
| 加载顺序 | **后加载**      | **先加载**                          |
| 所属模块 | Spring Boot     | Spring Cloud                        |
| 典型用途 | 普通业务配置    | 配置中心（Config Server）、加密密钥 |
| 是否必须 | 是（至少一个）  | 仅 Spring Cloud 项目需要            |

**加载顺序示例：**
```
bootstrap.yml → application.yml → application-{profile}.yml
```

> 在非 Spring Cloud 项目中，`bootstrap.yml` 不是必须的，甚至可能不会加载。

---

## 15、.properties 和 .yml 配置文件有什么区别？

**满分答案：**

| 对比项     | .properties                        | .yml / .yaml                       |
| ---------- | ---------------------------------- | ---------------------------------- |
| 语法       | `key=value`，扁平结构              | 树形缩进，层级结构                 |
| 可读性     | 扁平结构，多层级时**可读性差**     | 树形结构，**可读性好**             |
| 中文支持   | 默认 ISO-8859-1，中文需转 Unicode  | **原生 UTF-8**，中文无乱码         |
| 列表/数组  | 写法冗长：`list[0]=a, list[1]=b`   | 简洁：`- a \n  - b`                |
| Map 结构   | 写法冗长                           | 原生支持，简洁                     |
| 同时存在   | 同路径下 `.properties` 优先级**更高** | 优先级低于 .properties           |

**示例对比：**
```properties
# properties
server.port=8080
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
```

```yaml
# yml (更简洁清晰)
server:
  port: 8080
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
```

> **建议**：生产项目统一使用 `.yml` 格式。

---

## 16、Spring Boot 多环境配置如何实现？

**满分答案：**

Spring Boot 通过 **`spring.profiles.active`** 实现多环境配置。

**方式一：多个配置文件（推荐）**
```
application.yml              # 公共配置
application-dev.yml           # 开发环境
application-test.yml          # 测试环境
application-prod.yml          # 生产环境
```

在 `application.yml` 中指定：
```yaml
spring:
  profiles:
    active: dev   # 指定环境
```

**方式二：单文件多文档块**
```yaml
# application.yml
spring:
  profiles:
    active: dev

---
spring:
  config:
    activate:
      on-profile: dev
server:
  port: 8081

---
spring:
  config:
    activate:
      on-profile: prod
server:
  port: 80
```

**启动时切换环境：**
```bash
# 命令行参数（优先级最高）
java -jar app.jar --spring.profiles.active=prod

# 环境变量
export SPRING_PROFILES_ACTIVE=prod

# .mvn/jvm.config（Maven 启动时）
-Dspring.profiles.active=prod
```

**@Profile 注解：**
```java
@Component
@Profile("dev")  // 仅在 dev 环境生效
public class DevDataInitializer implements CommandLineRunner { }
```

---

## 17、@Value 和 @ConfigurationProperties 的区别？

**满分答案：**

| 对比项         | @Value                             | @ConfigurationProperties                |
| -------------- | ---------------------------------- | --------------------------------------- |
| 绑定方式       | **逐个**属性绑定                   | **批量**绑定（按前缀）                  |
| 松散绑定       | ❌ 不支持（如 `max-conn` ≈ `maxConn`） | ✅ **支持**                            |
| 属性校验       | ❌ 需手动配合 `@Validated`         | ✅ **天然支持 JSR303 校验**             |
| 复杂类型       | ❌ 不支持 List/Map 直接注入        | ✅ **原生支持 List、Map、对象嵌套**     |
| SpEL 表达式    | ✅ **支持**（如 `#{2*3}`）         | ❌ 不支持                               |
| 适用场景       | 注入少量配置                       | **注入一组相关配置**                    |

**示例对比：**
```java
// @Value：逐个注入
@Component
public class AppConfig {
    @Value("${app.name}")
    private String name;
    
    @Value("${app.max-connections:100}")  // 默认值
    private int maxConnections;
}

// @ConfigurationProperties：批量注入
@Component
@ConfigurationProperties(prefix = "app")
@Validated
public class AppProperties {
    @NotNull
    private String name;
    
    @Min(1) @Max(1000)
    private int maxConnections = 100;  // 默认值
    
    private List<String> whitelist;
    private Map<String, Integer> limits;
    // getter/setter 必须提供
}
```

> **官方推荐**：一组相关配置使用 `@ConfigurationProperties`，单个配置使用 `@Value`。

---

## 18、Spring Boot 中如何配置拦截器？

**满分答案：**

**实现步骤：**

**第 1 步：定义拦截器**
```java
@Component
public class AuthInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, 
                             HttpServletResponse response, 
                             Object handler) {
        // 前置处理：鉴权逻辑
        String token = request.getHeader("Authorization");
        if (token == null) {
            throw new UnauthorizedException("未登录");
        }
        return true;  // true=放行，false=拦截
    }
    
    @Override
    public void postHandle(HttpServletRequest request, 
                           HttpServletResponse response, 
                           Object handler,
                           ModelAndView modelAndView) {
        // 后置处理（Controller 执行后、视图渲染前）
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request, 
                                HttpServletResponse response, 
                                Object handler, Exception ex) {
        // 请求完成后（视图渲染后）
    }
}
```

**第 2 步：注册拦截器**
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Autowired
    private AuthInterceptor authInterceptor;
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(authInterceptor)
            .addPathPatterns("/api/**")        // 拦截路径
            .excludePathPatterns("/api/login", "/api/register"); // 排除路径
    }
}
```

**拦截器 vs 过滤器：**

| 对比项   | Interceptor（拦截器）   | Filter（过滤器）                 |
| -------- | ----------------------- | -------------------------------- |
| 容器     | Spring MVC 容器         | Servlet 容器                     |
| 粒度     | 可访问 Handler（Controller） | 只可访问 Request/Response    |
| 生命周期 | Spring 管理（可注入 Bean） | Servlet 管理                   |
| 执行顺序 | Filter → Interceptor → Controller | 最先执行（最外层）         |

---

## 19、Spring Boot 中如何解决跨域问题？

**满分答案：**

CORS（跨域资源共享）问题在前端分离架构中必然遇到。Spring Boot 提供了 **3 种** 解决方案：

| 方案                       | 适用场景               |
| -------------------------- | ---------------------- |
| `@CrossOrigin`             | 单个 Controller/接口   |
| `WebMvcConfigurer`         | 全局配置               |
| `CorsFilter`               | 更底层，与 Spring Security 配合 |

**方案一：@CrossOrigin（局部，最简单）**
```java
@RestController
@CrossOrigin(origins = "http://localhost:3000", maxAge = 3600)
public class UserController { }
```

**方案二：全局跨域配置（最常用）**
```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")                    // 所有路径
            .allowedOrigins("http://localhost:3000")   // 允许的来源
            .allowedMethods("GET", "POST", "PUT", "DELETE", "OPTIONS")
            .allowedHeaders("*")                       // 允许的请求头
            .allowCredentials(true)                    // 允许携带 Cookie
            .maxAge(3600);                             // 预检请求缓存时间
    }
}
```

**方案三：CorsFilter**
```java
@Bean
public CorsFilter corsFilter() {
    CorsConfiguration config = new CorsConfiguration();
    config.addAllowedOrigin("http://localhost:3000");
    config.addAllowedMethod("*");
    config.addAllowedHeader("*");
    config.setAllowCredentials(true);
    
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", config);
    return new CorsFilter(source);
}
```

---

## 20、Spring Boot 如何实现定时任务？

**满分答案：**

**第 1 步：开启定时任务**
```java
@SpringBootApplication
@EnableScheduling  // 关键注解
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

**第 2 步：定义定时任务**
```java
@Component
public class ReportTask {
    
    // Cron 表达式：每天凌晨 2 点执行
    @Scheduled(cron = "0 0 2 * * ?")
    public void generateDailyReport() {
        System.out.println("生成日报");
    }
    
    // 固定间隔：上一次执行完成后 5 秒再执行
    @Scheduled(fixedDelay = 5000)
    public void processQueue() { }
    
    // 固定频率：每 10 秒执行一次（即使上一次未完成）
    @Scheduled(fixedRate = 10000)
    public void checkStatus() { }
}
```

**Cron 表达式格式（7 位）：**
```
秒 分 时 日 月 星期 年（年可选）
```

| 常用 Cron                | 含义                   |
| ------------------------ | ---------------------- |
| `0 0 2 * * ?`            | 每天凌晨 2 点          |
| `0 0/30 * * * ?`         | 每 30 分钟             |
| `0 0 9-18 ? * MON-FRI`   | 工作日 9:00~18:00      |
| `0 0 1 1 * ?`            | 每月 1 日凌晨 1 点     |

---

## 21、Spring Boot 如何实现异步任务？

**满分答案：**

**第 1 步：开启异步支持**
```java
@SpringBootApplication
@EnableAsync  // 关键注解
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

**第 2 步：定义异步方法**
```java
@Service
public class NotificationService {
    
    @Async
    public void sendEmail(String to, String content) {
        // 该方法会在独立线程中执行，不会阻塞主线程
        System.out.println("发送邮件: " + Thread.currentThread().getName());
    }
    
    // 返回 Future
    @Async
    public CompletableFuture<String> processFile(String path) {
        // 耗时处理
        return CompletableFuture.completedFuture("处理完成");
    }
}
```

**第 3 步（可选）：自定义线程池**
```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);           // 核心线程数
        executor.setMaxPoolSize(20);           // 最大线程数
        executor.setQueueCapacity(100);        // 队列容量
        executor.setRejectedExecutionHandler(new CallerRunsPolicy());
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}
```

---

## 22、Spring Boot 如何进行数据校验？

**满分答案：**

Spring Boot 默认集成了 **Hibernate Validator**（JSR303 的实现），通过注解即可完成校验。

**常用校验注解：**

| 注解              | 说明         |
| ----------------- | ------------ |
| `@NotNull`        | 不能为 null  |
| `@NotEmpty`       | 不能为 null + 空字符串（或空集合） |
| `@NotBlank`       | 不能为 null + 空字符串 + 纯空格   |
| `@Size(min, max)` | 长度范围     |
| `@Min` / `@Max`   | 最小/最大值  |
| `@Email`          | 邮箱格式     |
| `@Pattern(regexp)`| 正则匹配     |
| `@Range(min, max)`| 数值范围     |
| `@Positive`       | 必须为正数   |

**使用方式：**
```java
// 1. 实体类加校验注解
public class UserDTO {
    @NotBlank(message = "用户名不能为空")
    private String username;
    
    @Size(min = 6, max = 20, message = "密码长度6-20位")
    private String password;
    
    @Email(message = "邮箱格式不正确")
    private String email;
    
    @Min(value = 18, message = "年龄不能小于18")
    @Max(value = 120, message = "年龄不能大于120")
    private Integer age;
}

// 2. Controller 中加 @Valid
@PostMapping("/register")
public Result register(@Valid @RequestBody UserDTO user, BindingResult bindingResult) {
    if (bindingResult.hasErrors()) {
        return Result.error(bindingResult.getAllErrors().get(0).getDefaultMessage());
    }
    // 业务逻辑
}
```

> 通过 `@ControllerAdvice` 全局异常处理可以统一处理校验异常（参考第 10 题）。

---

## 23、Spring Boot 如何配置日志？

**满分答案：**

Spring Boot 默认使用 **Logback** 作为日志框架（内嵌了 `logback-classic`），同时也支持 Log4j2。

**application.yml 基础配置：**
```yaml
logging:
  level:
    root: INFO                    # 根日志级别
    com.example.dao: DEBUG        # 指定包级别
    org.springframework: WARN     # Spring 框架日志级别
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
  file:
    name: logs/app.log            # 写入文件
    max-size: 100MB               # 单文件最大大小
    max-history: 30               # 保留天数
```

**日志级别从低到高：**
```
TRACE < DEBUG < INFO < WARN < ERROR < FATAL
```

**注意事项：**
- 配置文件命名必须使用 `logback-spring.xml`（而非 `logback.xml`），才能使用 Spring 的 `<springProfile>` 标签
- 如果使用 Log4j2，需排除 Logback 并引入 `spring-boot-starter-log4j2`

---

## 24、Spring Boot 如何进行单元测试？

**满分答案：**

Spring Boot 通过 `spring-boot-starter-test` 提供测试支持，内部整合了 JUnit 5、Mockito、AssertJ 等。

**测试示例：**
```java
@SpringBootTest  // 启动完整 Spring 上下文
class UserServiceTest {
    
    @MockBean  // 替换容器中的 Bean 为 Mock 对象
    private UserDao userDao;
    
    @Autowired
    private UserService userService;
    
    @Test
    void testGetUser() {
        // 1. 预设行为
        User mockUser = new User(1L, "张三");
        Mockito.when(userDao.findById(1L)).thenReturn(mockUser);
        
        // 2. 执行测试
        User result = userService.getUser(1L);
        
        // 3. 断言验证
        assertEquals("张三", result.getName());
        Mockito.verify(userDao).findById(1L);  // 验证方法被调用
    }
}
```

**常用测试注解：**

| 注解              | 作用                                             |
| ----------------- | ------------------------------------------------ |
| `@SpringBootTest` | 启动完整 Spring 上下文（慢，用于集成测试）       |
| `@WebMvcTest`     | 仅加载 MVC 层（Controller），适合 Controller 单元测试 |
| `@DataJpaTest`    | 仅加载 JPA 层                                   |
| `@MockBean`       | 创建 Mock 对象并替换 Spring 容器中的 Bean        |
| `@SpyBean`        | 部分 Mock，部分真实调用                          |
| `@Test`           | JUnit 5 标记测试方法                             |
| `@BeforeEach`     | 每个测试方法前执行                               |

---

## 25、如何获取 Spring 容器中的 ApplicationContext？

**满分答案：**

| 方式                             | 说明                       |
| -------------------------------- | -------------------------- |
| `@Autowired` 注入                | 直接注入到 Bean 中         |
| 实现 `ApplicationContextAware`   | 静态持有（工具类最常用）   |
| 主类中获取                       | `SpringApplication.run()` 返回值 |
| `@EventListener` 监听就绪事件    | 容器完全就绪后获取         |

**最常用方式（工具类）：**
```java
@Component
public class SpringContextHolder implements ApplicationContextAware {
    private static ApplicationContext context;
    
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) {
        context = applicationContext;
    }
    
    public static <T> T getBean(Class<T> clazz) {
        return context.getBean(clazz);
    }
    
    public static Object getBean(String name) {
        return context.getBean(name);
    }
}
```

---

## 26、如何排除/禁用特定的自动配置？

**满分答案：**

| 方式                         | 示例                                                   |
| ---------------------------- | ------------------------------------------------------ |
| `@SpringBootApplication` 排除 | `@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)` |
| `@EnableAutoConfiguration` 排除 | `@EnableAutoConfiguration(exclude = {...})`            |
| 配置文件                     | `spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration` |
| 条件注解                    | 用 `@ConditionalOnMissingBean` 自定义替代 Bean         |

---

## 27、Spring Boot 的启动流程是怎样的？

**满分答案（核心 7 步）：**

1. **创建 SpringApplication 实例**：推断应用类型（Servlet/Reactive/非Web）
2. **获取 SpringApplicationRunListeners**：发布启动事件
3. **解析命令行参数**：准备 `ApplicationArguments`
4. **准备 Environment**：加载配置文件（application.yml 等）
5. **打印 Banner**：`SpringApplicationBannerPrinter`
6. **创建 ApplicationContext**：根据应用类型创建（AnnotationConfigServletWebServerApplicationContext 等）
7. **刷新 ApplicationContext**：`refresh()` → 自动配置 → 内嵌容器启动 → 发布 `ApplicationReadyEvent`

```
new SpringApplication() → RunListener.starting()
→ 准备 Environment → 准备 ApplicationContext
→ refresh() → afterRefresh()
→ RunListener.started() → callRunners()
→ RunListener.ready()
```

---

## 28、如何集成 Spring Boot 和 MySQL？

**满分答案：**

**第 1 步：添加依赖**
```xml
<!-- Spring Boot JDBC Starter（含 HikariCP 连接池） -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>

<!-- MySQL 驱动 -->
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
</dependency>

<!-- 推荐：使用 MyBatis-Plus 或 JPA 代替 JdbcTemplate -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.3</version>
</dependency>
```

**第 2 步：配置 application.yml**
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb?useSSL=false&serverTimezone=Asia/Shanghai&characterEncoding=utf8
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver
    # HikariCP 连接池配置（可选）
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      connection-timeout: 30000

# MyBatis-Plus 配置（可选）
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl  # 输出 SQL
  mapper-locations: classpath*:/mapper/**/*.xml
```

**第 3 步：使用 JdbcTemplate**
```java
@Repository
public class UserDao {
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    public User findById(Long id) {
        String sql = "SELECT * FROM user WHERE id = ?";
        return jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<>(User.class), id);
    }
}
```

**第 4 步：事务管理（自动配置，直接使用 `@Transactional`）**
```java
@Service
public class UserService {
    @Transactional
    public void createUser(User user) {
        // 数据库操作，异常时自动回滚
    }
}
```

---

## 29、Spring Boot 如何实现接口防刷/限流？

**满分答案：**

防刷/限流在保护接口不被恶意调用方面非常重要，主要有以下几种方案：

| 方案               | 说明                                     | 适用场景           |
| ------------------ | ---------------------------------------- | ------------------ |
| **自定义拦截器**   | Redis 记录 IP+接口+时间窗口请求次数      | 简单防刷           |
| **Guava RateLimiter** | 单机令牌桶限流                        | 单体应用           |
| **Sentinel**       | 分布式限流 + 可视化控制台（推荐）        | 生产级微服务       |
| **Bucket4j**       | 分布式令牌桶，支持 JCache/Hazelcast/Redis | 需要分布式限流     |
| **Gateway 限流**   | 在网关层统一限流                         | 网关统一入口       |

**自定义拦截器 + Redis 示例：**
```java
@Component
public class RateLimitInterceptor implements HandlerInterceptor {
    @Autowired
    private RedisTemplate<String, Integer> redisTemplate;
    
    @Override
    public boolean preHandle(HttpServletRequest request, 
                             HttpServletResponse response, 
                             Object handler) {
        String key = "rate_limit:" + request.getRemoteAddr() + ":" + request.getRequestURI();
        Integer count = redisTemplate.opsForValue().get(key);
        
        if (count == null) {
            // 第一次请求，设置窗口计数
            redisTemplate.opsForValue().set(key, 1, Duration.ofSeconds(60));
            return true;
        }
        
        if (count > 100) {  // 每分钟最多 100 次
            throw new RateLimitException("请求过于频繁，请稍后再试");
        }
        
        redisTemplate.opsForValue().increment(key);
        return true;
    }
}
```

---

## 30、Spring Boot 的 Jar 包和普通 Jar 包有什么不同？

**满分答案：**

| 对比项     | Spring Boot Fat Jar                 | 普通 Jar                 |
| ---------- | ----------------------------------- | ------------------------ |
| 依赖包含   | **包含所有依赖**                    | 仅包含自身代码           |
| 运行方式   | `java -jar xxx.jar`                 | 需指定 classpath         |
| 文件结构   | BOOT-INF/lib（依赖Jar） + BOOT-INF/classes（自身代码） | 仅 class 文件 |
| 类加载器   | **自定义 LaunchedURLClassLoader**   | 标准 URLClassLoader      |
| 是否独立   | ✅ **可独立运行**                    | ❌ 依赖外部 classpath    |

**Fat Jar 目录结构：**
```
myapp.jar
├── META-INF/
│   └── MANIFEST.MF          # Main-Class: JarLauncher
├── org/springframework/boot/loader/  # Spring Boot 类加载器
├── BOOT-INF/
│   ├── classes/             # 自身代码
│   └── lib/                 # 依赖 Jar 包
```

---

## 31、Spring Boot 的 Banner 如何自定义？

**满分答案：**

**方式一：banner.txt**
在 `src/main/resources/` 下创建 `banner.txt`，放入自定义文字/图案，启动时会自动打印。

**方式二：banner.gif/png/jpg**
在 `src/main/resources/` 下放入 `banner.gif`，启动时会在控制台打印图片（需 ANSI 支持）。

**方式三：关闭 Banner**
```java
SpringApplication app = new SpringApplication(MyApplication.class);
app.setBannerMode(Banner.Mode.OFF);
app.run(args);
```

或者在 `application.yml` 中：
```yaml
spring:
  main:
    banner-mode: off
```

---

[//]: # "以上内容涵盖了 Spring Boot 面试中最常见的约 31 个核心问题，包含自动配置原理、Starter 机制、数据校验、多环境、异步任务、定时任务、全局异常处理、拦截器、跨域、单元测试、启动流程等。"
